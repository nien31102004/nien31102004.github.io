---
title : "Tạo Lambda Decision Engine"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.7.1 </b> "
---

Lambda Decision Engine chịu trách nhiệm tổng hợp kết quả quét mã độc và phân tích AI để xác định trạng thái cuối cùng của tài liệu.

Mở [AWS Lambda console](https://console.aws.amazon.com/lambda/) và chọn **Create function**.

Trong phần cấu hình Lambda, nhập:

+ **Function name**: `SecureDocDecisionEngine`
+ **Runtime**: `Python 3.14`
+ **Architecture**: `x86_64`
+ **Execution role**: Chọn role có quyền truy cập Amazon S3, Amazon DynamoDB và CloudWatch Logs.

Sau đó chọn **Create function**.

![Configure function](/images/5-Workshop/5.7/5.7.1/5.7.1.1.png)

Trong phần **Code source**, thay nội dung file `lambda_function.py` bằng mã nguồn Decision Engine của dự án và chọn **Deploy**.
```text
import json
import os
import traceback
import urllib.parse
from datetime import datetime, timezone
from decimal import Decimal, InvalidOperation
from typing import Any

import boto3
from botocore.exceptions import ClientError

CODE_VERSION = "secure-doc-decision-engine-admin-pending-v2"


# =========================================================
# AWS clients và cấu hình
# =========================================================

s3_client = boto3.client("s3")
dynamodb = boto3.resource("dynamodb")

TABLE_NAME = os.environ.get(
    "DOCUMENTS_TABLE",
    "SecureDocuments",
).strip()

DOCUMENT_BUCKET = os.environ.get(
    "DOCUMENT_BUCKET",
    "secure-ai-document-storage",
).strip()

QUARANTINE_PREFIX = os.environ.get(
    "QUARANTINE_PREFIX",
    "quarantine",
).strip().strip("/")

CLEAN_PREFIX = os.environ.get(
    "CLEAN_PREFIX",
    "clean",
).strip().strip("/")

REJECT_PREFIX = os.environ.get(
    "REJECT_PREFIX",
    "reject",
).strip().strip("/")

ADMIN_GROUP = os.environ.get(
    "ADMIN_GROUP",
    "admin",
).strip()

CORS_ORIGIN = os.environ.get(
    "CORS_ORIGIN",
    "*",
).strip()

table = dynamodb.Table(TABLE_NAME)

VALID_DECISION_SOURCES = {
    "SYSTEM_RULES",
    "ADMIN_REVIEW",
}

VALID_SYSTEM_ACTIONS = {
    "ALLOW",
    "REVIEW",
    "REJECT",
}

VALID_ADMIN_ACTIONS = {
    "ALLOW",
    "REJECT",
}

ADMIN_REVIEW_STATUSES = {
    "MANUAL_REVIEW",
    "DECISION_PENDING",
}

SYSTEM_SOURCE_STATUSES = {
    "AI_ANALYSIS_COMPLETED",
    "DECISION_PENDING",
    "MALWARE_DETECTED",
    "MALWARE_SCAN_COMPLETED",
}

REVIEW_SOURCE_STATUSES = {
    "AI_ANALYSIS_COMPLETED",
    "MALWARE_SCAN_COMPLETED",
    "MANUAL_REVIEW",
    "DECISION_PENDING",
    "SCAN_ERROR",
}


class ForbiddenError(Exception):
    """Người gọi đã xác thực nhưng không có quyền thực hiện thao tác."""


# =========================================================
# Lambda handler
# =========================================================

def lambda_handler(event: Any, context: Any) -> dict:
    print(f"CODE_VERSION: {CODE_VERSION}")
    print("Received Decision Engine event:")
    print(json.dumps(event, default=str, indent=2))

    try:
        normalized_event = normalize_event(event)
        document_id = get_document_id(normalized_event)
        decision_source = get_decision_source(normalized_event)
        requested_action = normalized_event.get("action")

        if decision_source == "ADMIN_REVIEW":
            validate_internal_admin_review(event)

        item = get_document(document_id)

        if not item:
            return api_response(
                404,
                {
                    "message": "Không tìm thấy tài liệu",
                    "documentId": document_id,
                },
            )

        print_document_context(item)

        action = resolve_final_action(
            item=item,
            decision_source=decision_source,
            requested_action=requested_action,
        )

        print(
            "Resolved decision:",
            json.dumps(
                {
                    "documentId": document_id,
                    "decisionSource": decision_source,
                    "resolvedAction": action,
                    "malwareStatus": item.get("malwareStatus"),
                    "systemRecommendedAction": item.get(
                        "systemRecommendedAction"
                    ),
                },
                default=str,
            ),
        )

        if action == "ALLOW":
            result = process_allow(
                item=item,
                decision_source=decision_source,
            )
        elif action == "REJECT":
            result = process_reject(
                item=item,
                decision_source=decision_source,
            )
        else:
            result = process_review(
                item=item,
                decision_source=decision_source,
            )

        print(
            "Decision completed:",
            json.dumps(result, default=str, indent=2),
        )

        return api_response(200, result)

    except ForbiddenError as error:
        print(f"Authorization error: {error}")
        return api_response(403, {"message": str(error)})

    except ValueError as error:
        print(f"Validation error: {error}")
        return api_response(400, {"message": str(error)})

    except ClientError as error:
        print("AWS ClientError:")
        print(json.dumps(error.response, default=str, indent=2))

        error_code = (
            error.response
            .get("Error", {})
            .get("Code", "Unknown")
        )

        if error_code == "ConditionalCheckFailedException":
            return api_response(
                409,
                {
                    "message": (
                        "Tài liệu đã được xử lý hoặc "
                        "không còn ở trạng thái hợp lệ"
                    )
                },
            )

        return api_response(
            500,
            {
                "message": "AWS service error",
                "errorCode": error_code,
            },
        )

    except Exception as error:
        print(f"Unexpected error: {error}")
        traceback.print_exc()
        return api_response(
            500,
            {"message": "Internal server error"},
        )


# =========================================================
# Chuẩn hóa event và phân quyền
# =========================================================

def normalize_event(event: Any) -> dict:
    """
    Hỗ trợ:
    1. Lambda-to-Lambda event trực tiếp.
    2. API Gateway event có body JSON.
    """
    if not isinstance(event, dict):
        raise ValueError("Event phải là JSON object")

    body = event.get("body")

    if body is None:
        return dict(event)

    if isinstance(body, dict):
        parsed_body = body
    elif isinstance(body, str):
        try:
            parsed_body = json.loads(body)
        except json.JSONDecodeError as error:
            raise ValueError(
                "Request body không phải JSON hợp lệ"
            ) from error
    else:
        raise ValueError("Request body không hợp lệ")

    if not isinstance(parsed_body, dict):
        raise ValueError("Request body phải là JSON object")

    merged = dict(event)
    merged.update(parsed_body)
    return merged


def get_document_id(event: dict) -> str:
    document_id = event.get("documentId")

    if not document_id:
        path_parameters = event.get("pathParameters") or {}
        document_id = path_parameters.get("documentId")

    document_id = str(document_id or "").strip()

    if not document_id:
        raise ValueError("documentId là bắt buộc")

    return document_id


def get_decision_source(event: dict) -> str:
    decision_source = str(
        event.get("decisionSource", "SYSTEM_RULES")
    ).strip().upper()

    if decision_source not in VALID_DECISION_SOURCES:
        raise ValueError(
            "decisionSource phải là SYSTEM_RULES hoặc ADMIN_REVIEW"
        )

    return decision_source


def require_admin(event: dict) -> None:
    claims = extract_authorizer_claims(event)
    groups = parse_cognito_groups(
        claims.get("cognito:groups")
    )

    if ADMIN_GROUP not in groups:
        raise ForbiddenError(
            f"Chỉ thành viên nhóm Cognito '{ADMIN_GROUP}' "
            "được thực hiện admin review"
        )


def extract_authorizer_claims(event: dict) -> dict:
    request_context = event.get("requestContext") or {}
    authorizer = request_context.get("authorizer") or {}

    # HTTP API JWT authorizer.
    jwt_claims = (
        authorizer.get("jwt", {}).get("claims")
        if isinstance(authorizer.get("jwt"), dict)
        else None
    )

    if isinstance(jwt_claims, dict):
        return jwt_claims

    # REST API Cognito authorizer.
    claims = authorizer.get("claims")
    if isinstance(claims, dict):
        return claims

    return {}


def parse_cognito_groups(raw_groups: Any) -> set[str]:
    if isinstance(raw_groups, list):
        return {
            str(group).strip()
            for group in raw_groups
            if str(group).strip()
        }

    if not raw_groups:
        return set()

    text = str(raw_groups).strip()

    if text.startswith("["):
        try:
            parsed = json.loads(text)
            if isinstance(parsed, list):
                return {
                    str(group).strip()
                    for group in parsed
                    if str(group).strip()
                }
        except json.JSONDecodeError:
            pass

    return {
        group.strip()
        for group in text.split(",")
        if group.strip()
    }


# =========================================================
# DynamoDB
# =========================================================

def get_document(document_id: str) -> dict | None:
    response = table.get_item(
        Key={"documentId": document_id},
        ConsistentRead=True,
    )
    return response.get("Item")


def print_document_context(item: dict) -> None:
    print(
        "Document context:",
        json.dumps(
            {
                "documentId": item.get("documentId"),
                "status": item.get("status"),
                "bucket": item.get("bucket"),
                "s3Key": item.get("s3Key"),
                "currentPrefix": item.get("currentPrefix"),
                "malwareStatus": item.get("malwareStatus"),
                "aiStatus": item.get("aiStatus"),
                "aiRecommendedAction": item.get(
                    "aiRecommendedAction"
                ),
                "systemRecommendedAction": item.get(
                    "systemRecommendedAction"
                ),
                "finalDecision": item.get("finalDecision"),
            },
            default=str,
        ),
    )


# =========================================================
# Quyết định
# =========================================================

def resolve_final_action(
    item: dict,
    decision_source: str,
    requested_action: Any,
) -> str:
    """
    Thứ tự ưu tiên:
    1. GuardDuty phát hiện malware luôn REJECT.
    2. Admin chỉ ALLOW/REJECT tài liệu đang chờ review.
    3. System đọc systemRecommendedAction khi AI hoàn thành.
    4. GuardDuty hoặc AI chưa hoàn thành thì REVIEW.
    """
    malware_status = normalized_upper(
        item.get("malwareStatus"),
        "UNKNOWN",
    )
    current_status = normalized_upper(
        item.get("status"),
        "UNKNOWN",
    )

    if malware_status == "THREATS_FOUND":
        return "REJECT"

    if decision_source == "ADMIN_REVIEW":
        return resolve_admin_action(
            requested_action=requested_action,
            malware_status=malware_status,
            current_status=current_status,
        )

    return resolve_system_action(
        item=item,
        malware_status=malware_status,
    )


def resolve_admin_action(
    requested_action: Any,
    malware_status: str,
    current_status: str,
) -> str:
    action = normalized_upper(requested_action)

    if action not in VALID_ADMIN_ACTIONS:
        raise ValueError(
            "Admin action phải là ALLOW hoặc REJECT"
        )

    if current_status not in {
        "MANUAL_REVIEW",
        "DECISION_PENDING",
    }:
        raise ValueError(
            "Tài liệu không ở trạng thái "
            "MANUAL_REVIEW hoặc DECISION_PENDING"
        )

    if (
        action == "ALLOW"
        and malware_status != "NO_THREATS_FOUND"
    ):
        raise ValueError(
            "Không thể approve tài liệu chưa được "
            "GuardDuty xác nhận NO_THREATS_FOUND"
        )

    return action


def resolve_system_action(
    item: dict,
    malware_status: str,
) -> str:
    if malware_status != "NO_THREATS_FOUND":
        return "REVIEW"

    ai_status = normalized_upper(
        item.get("aiStatus"),
        "UNKNOWN",
    )

    if ai_status != "COMPLETED":
        return "REVIEW"

    action = normalized_upper(
        item.get("systemRecommendedAction"),
        "REVIEW",
    )

    if action not in VALID_SYSTEM_ACTIONS:
        return "REVIEW"

    return action


# =========================================================
# ALLOW / REJECT / REVIEW
# =========================================================

def process_allow(
    item: dict,
    decision_source: str,
) -> dict:
    validate_allow_preconditions(item)

    return process_move(
        item=item,
        decision_source=decision_source,
        destination_prefix=CLEAN_PREFIX,
        moving_status="MOVING_TO_CLEAN",
        final_status="SAFE",
        final_decision="ALLOW",
    )


def process_reject(
    item: dict,
    decision_source: str,
) -> dict:
    return process_move(
        item=item,
        decision_source=decision_source,
        destination_prefix=REJECT_PREFIX,
        moving_status="MOVING_TO_REJECT",
        final_status="REJECTED",
        final_decision="REJECT",
    )


def process_move(
    item: dict,
    decision_source: str,
    destination_prefix: str,
    moving_status: str,
    final_status: str,
    final_decision: str,
) -> dict:
    document_id = item["documentId"]

    if is_already_completed(
        item=item,
        status=final_status,
        prefix=destination_prefix,
        decision=final_decision,
    ):
        cleanup_status = retry_pending_source_cleanup(item)
        return {
            "documentId": document_id,
            "status": final_status,
            "finalDecision": final_decision,
            "message": "Tài liệu đã được xử lý trước đó",
            "s3Key": item.get("s3Key"),
            "sourceCleanupStatus": cleanup_status,
        }

    source_key = get_valid_source_key(item)
    source_version_id = get_source_version_id(item)
    destination_key = build_destination_key(
        source_key=source_key,
        destination_prefix=destination_prefix,
    )
    previous_status = normalized_upper(
        item.get("status"),
        "DECISION_PENDING",
    )

    allowed_statuses = get_allowed_source_statuses(
        decision_source
    ) | {moving_status}

    mark_moving(
        document_id=document_id,
        target_status=moving_status,
        expected_statuses=allowed_statuses,
        source_key=source_key,
        destination_key=destination_key,
        source_version_id=source_version_id,
    )

    try:
        destination_version_id = copy_object_and_verify(
            source_key=source_key,
            destination_key=destination_key,
            source_version_id=source_version_id,
        )

        finalize_document_move(
            item=item,
            source_key=source_key,
            source_version_id=source_version_id,
            destination_key=destination_key,
            destination_version_id=destination_version_id,
            moving_status=moving_status,
            final_status=final_status,
            final_decision=final_decision,
            final_prefix=destination_prefix,
            decision_source=decision_source,
        )
    except Exception as error:
        rollback_move_status(
            document_id=document_id,
            moving_status=moving_status,
            previous_status=previous_status,
            error_message=str(error),
        )
        raise

    cleanup_status = cleanup_source_after_move(
        document_id=document_id,
        source_key=source_key,
        source_version_id=source_version_id,
    )

    return {
        "documentId": document_id,
        "status": final_status,
        "finalDecision": final_decision,
        "finalDecisionSource": decision_source,
        "currentPrefix": destination_prefix,
        "s3Key": destination_key,
        "destinationVersionId": destination_version_id,
        "sourceCleanupStatus": cleanup_status,
    }


def validate_allow_preconditions(item: dict) -> None:
    malware_status = normalized_upper(
        item.get("malwareStatus"),
        "UNKNOWN",
    )

    if malware_status != "NO_THREATS_FOUND":
        raise ValueError(
            "Không thể chuyển sang clean vì "
            "GuardDuty chưa xác nhận an toàn"
        )


def process_review(
    item: dict,
    decision_source: str,
) -> dict:
    document_id = item["documentId"]
    source_key = get_valid_source_key(item)
    now = utc_now()

    if (
        normalized_upper(item.get("status")) == "MANUAL_REVIEW"
        and normalized_upper(item.get("reviewStatus")) == "PENDING"
    ):
        return {
            "documentId": document_id,
            "status": "MANUAL_REVIEW",
            "reviewStatus": "PENDING",
            "finalDecision": "PENDING",
            "currentPrefix": QUARANTINE_PREFIX,
            "s3Key": source_key,
            "message": "Tài liệu đã nằm trong hàng chờ review",
        }

    response = table.update_item(
        Key={"documentId": document_id},
        UpdateExpression="""
            SET #status = :manualReview,
                reviewStatus = :reviewPending,
                finalDecision = :finalPending,
                finalDecisionSource = :source,
                currentPrefix = :quarantinePrefix,
                reviewRequestedAt = :reviewRequestedAt,
                updatedAt = :updatedAt
        """,
        ConditionExpression="""
            attribute_exists(documentId)
            AND #status IN (
                :aiCompleted,
                :malwareCompleted,
                :manualReview,
                :decisionPending,
                :scanError
            )
        """,
        ExpressionAttributeNames={"#status": "status"},
        ExpressionAttributeValues={
            ":manualReview": "MANUAL_REVIEW",
            ":reviewPending": "PENDING",
            ":finalPending": "PENDING",
            ":source": decision_source,
            ":quarantinePrefix": QUARANTINE_PREFIX,
            ":reviewRequestedAt": now,
            ":updatedAt": now,
            ":aiCompleted": "AI_ANALYSIS_COMPLETED",
            ":malwareCompleted": "MALWARE_SCAN_COMPLETED",
            ":decisionPending": "DECISION_PENDING",
            ":scanError": "SCAN_ERROR",
        },
        ReturnValues="ALL_NEW",
    )

    attributes = response.get("Attributes", {})

    return {
        "documentId": document_id,
        "status": attributes.get("status", "MANUAL_REVIEW"),
        "reviewStatus": attributes.get(
            "reviewStatus",
            "PENDING",
        ),
        "finalDecision": attributes.get(
            "finalDecision",
            "PENDING",
        ),
        "currentPrefix": attributes.get(
            "currentPrefix",
            QUARANTINE_PREFIX,
        ),
        "s3Key": source_key,
    }


# =========================================================
# S3 move helpers
# =========================================================

def get_valid_source_key(item: dict) -> str:
    source_key = str(item.get("s3Key") or "").strip()

    if not source_key:
        raise ValueError("Document không có s3Key")

    decoded_key = urllib.parse.unquote_plus(source_key)
    expected_prefix = f"{QUARANTINE_PREFIX}/"

    if not decoded_key.startswith(expected_prefix):
        raise ValueError(
            "Object nguồn không nằm trong "
            f"{expected_prefix}: {decoded_key}"
        )

    return decoded_key


def build_destination_key(
    source_key: str,
    destination_prefix: str,
) -> str:
    parts = source_key.split("/", 1)

    if len(parts) != 2 or not parts[1]:
        raise ValueError(f"S3 key không hợp lệ: {source_key}")

    return f"{destination_prefix}/{parts[1]}"


def get_source_version_id(item: dict) -> str | None:
    version_id = (
        item.get("malwareObjectVersionId")
        or item.get("objectVersionId")
    )

    if version_id is None:
        return None

    normalized = str(version_id).strip()

    if normalized.upper() in {
        "",
        "NONE",
        "NULL",
        "UNKNOWN",
    }:
        return None

    return normalized


def copy_object_and_verify(
    source_key: str,
    destination_key: str,
    source_version_id: str | None,
) -> str | None:
    copy_source: dict[str, str] = {
        "Bucket": DOCUMENT_BUCKET,
        "Key": source_key,
    }
    source_head_parameters: dict[str, str] = {
        "Bucket": DOCUMENT_BUCKET,
        "Key": source_key,
    }

    if source_version_id:
        copy_source["VersionId"] = source_version_id
        source_head_parameters["VersionId"] = source_version_id

    source_head = s3_client.head_object(
        **source_head_parameters
    )
    source_size = source_head.get("ContentLength")

    print(
        "Copying object:",
        json.dumps(
            {
                "bucket": DOCUMENT_BUCKET,
                "sourceKey": source_key,
                "sourceVersionId": source_version_id,
                "destinationKey": destination_key,
                "sourceSize": source_size,
            },
            default=str,
        ),
    )

    response = s3_client.copy_object(
        Bucket=DOCUMENT_BUCKET,
        Key=destination_key,
        CopySource=copy_source,
        MetadataDirective="COPY",
        TaggingDirective="COPY",
    )

    print(
        "Copy response:",
        json.dumps(response, default=str, indent=2),
    )

    copy_result = response.get("CopyObjectResult", {})
    destination_etag = copy_result.get("ETag")
    destination_version_id = response.get("VersionId")

    if not destination_etag:
        raise RuntimeError("S3 CopyObject không trả ETag")

    verify_parameters: dict[str, str] = {
        "Bucket": DOCUMENT_BUCKET,
        "Key": destination_key,
    }

    if destination_version_id:
        verify_parameters["VersionId"] = destination_version_id

    verify_response = s3_client.head_object(
        **verify_parameters
    )
    verified_etag = verify_response.get("ETag")
    verified_size = verify_response.get("ContentLength")

    if not verified_etag:
        raise RuntimeError("Không xác minh được object đích")

    if destination_etag != verified_etag:
        raise RuntimeError(
            "ETag của object đích không khớp kết quả CopyObject"
        )

    if (
        source_size is not None
        and verified_size != source_size
    ):
        raise RuntimeError(
            "Kích thước object đích không khớp object nguồn"
        )

    print(
        "Destination verified:",
        json.dumps(
            {
                "destinationKey": destination_key,
                "eTag": verified_etag,
                "size": verified_size,
                "versionId": destination_version_id,
            },
            default=str,
        ),
    )

    return destination_version_id


def delete_source_object(
    source_key: str,
    source_version_id: str | None,
) -> None:
    parameters: dict[str, str] = {
        "Bucket": DOCUMENT_BUCKET,
        "Key": source_key,
    }

    if source_version_id:
        parameters["VersionId"] = source_version_id

    print(
        "Deleting source object:",
        json.dumps(
            {
                "bucket": DOCUMENT_BUCKET,
                "sourceKey": source_key,
                "versionId": source_version_id,
            },
            default=str,
        ),
    )

    s3_client.delete_object(**parameters)


# =========================================================
# Trạng thái move và hoàn tất DynamoDB
# =========================================================

def mark_moving(
    document_id: str,
    target_status: str,
    expected_statuses: set[str],
    source_key: str,
    destination_key: str,
    source_version_id: str | None,
) -> None:
    now = utc_now()
    expression_values: dict[str, Any] = {
        ":targetStatus": target_status,
        ":moveStartedAt": now,
        ":updatedAt": now,
        ":sourceKey": source_key,
        ":destinationKey": destination_key,
    }

    status_placeholders = []
    for index, status in enumerate(sorted(expected_statuses)):
        placeholder = f":expectedStatus{index}"
        expression_values[placeholder] = status
        status_placeholders.append(placeholder)

    update_expression = """
        SET #status = :targetStatus,
            moveStartedAt = :moveStartedAt,
            updatedAt = :updatedAt,
            moveSourceKey = :sourceKey,
            moveDestinationKey = :destinationKey
    """

    if source_version_id:
        update_expression += (
            ", moveSourceVersionId = :sourceVersionId"
        )
        expression_values[
            ":sourceVersionId"
        ] = source_version_id

    condition_expression = (
        "attribute_exists(documentId) "
        "AND #status IN ("
        + ", ".join(status_placeholders)
        + ")"
    )

    table.update_item(
        Key={"documentId": document_id},
        UpdateExpression=update_expression,
        ConditionExpression=condition_expression,
        ExpressionAttributeNames={"#status": "status"},
        ExpressionAttributeValues=expression_values,
    )


def rollback_move_status(
    document_id: str,
    moving_status: str,
    previous_status: str,
    error_message: str,
) -> None:
    try:
        now = utc_now()
        table.update_item(
            Key={"documentId": document_id},
            UpdateExpression="""
                SET #status = :previousStatus,
                    moveFailureReason = :failureReason,
                    moveFailedAt = :failedAt,
                    updatedAt = :updatedAt
            """,
            ConditionExpression="#status = :movingStatus",
            ExpressionAttributeNames={"#status": "status"},
            ExpressionAttributeValues={
                ":previousStatus": previous_status,
                ":failureReason": error_message[:1000],
                ":failedAt": now,
                ":updatedAt": now,
                ":movingStatus": moving_status,
            },
        )
        print(
            f"Rolled back {document_id} to {previous_status}"
        )
    except ClientError as rollback_error:
        print("Could not roll back moving status:")
        print(json.dumps(
            rollback_error.response,
            default=str,
            indent=2,
        ))


def finalize_document_move(
    item: dict,
    source_key: str,
    source_version_id: str | None,
    destination_key: str,
    destination_version_id: str | None,
    moving_status: str,
    final_status: str,
    final_decision: str,
    final_prefix: str,
    decision_source: str,
) -> None:
    document_id = item["documentId"]
    now = utc_now()
    risk_score = decimal_value(
        item.get(
            "systemRiskScore",
            item.get("riskScore", 0),
        )
    )
    risk_level = str(
        item.get(
            "systemRiskLevel",
            item.get("riskLevel", "UNKNOWN"),
        )
    ).strip().upper()

    expression_values: dict[str, Any] = {
        ":finalStatus": final_status,
        ":finalDecision": final_decision,
        ":decisionSource": decision_source,
        ":riskScore": risk_score,
        ":riskLevel": risk_level,
        ":prefix": final_prefix,
        ":destinationKey": destination_key,
        ":sourceKey": source_key,
        ":bucket": DOCUMENT_BUCKET,
        ":completedAt": now,
        ":updatedAt": now,
        ":movingStatus": moving_status,
        ":cleanupPending": "PENDING",
    }

    update_expression = """
        SET #status = :finalStatus,
            finalDecision = :finalDecision,
            finalDecisionSource = :decisionSource,
            finalRiskScore = :riskScore,
            finalRiskLevel = :riskLevel,
            currentPrefix = :prefix,
            s3Key = :destinationKey,
            originalSourceKey = :sourceKey,
            #bucket = :bucket,
            finalBucket = :bucket,
            completedAt = :completedAt,
            updatedAt = :updatedAt,
            sourceCleanupStatus = :cleanupPending
    """

    if source_version_id:
        update_expression += (
            ", originalSourceVersionId = :sourceVersionId"
        )
        expression_values[
            ":sourceVersionId"
        ] = source_version_id

    if destination_version_id:
        update_expression += (
            ", destinationVersionId = :destinationVersionId"
        )
        expression_values[
            ":destinationVersionId"
        ] = destination_version_id

    table.update_item(
        Key={"documentId": document_id},
        UpdateExpression=update_expression,
        ConditionExpression="""
            attribute_exists(documentId)
            AND #status = :movingStatus
        """,
        ExpressionAttributeNames={
            "#status": "status",
            "#bucket": "bucket",
        },
        ExpressionAttributeValues=expression_values,
    )


def cleanup_source_after_move(
    document_id: str,
    source_key: str,
    source_version_id: str | None,
) -> str:
    try:
        delete_source_object(
            source_key=source_key,
            source_version_id=source_version_id,
        )
        update_cleanup_status(
            document_id=document_id,
            status="COMPLETED",
        )
        return "COMPLETED"
    except ClientError as error:
        print("Source cleanup failed:")
        print(json.dumps(error.response, default=str, indent=2))
        update_cleanup_status(
            document_id=document_id,
            status="FAILED",
            error_message=str(error),
        )
        return "FAILED"


def retry_pending_source_cleanup(item: dict) -> str:
    cleanup_status = normalized_upper(
        item.get("sourceCleanupStatus"),
        "COMPLETED",
    )

    if cleanup_status == "COMPLETED":
        return "COMPLETED"

    source_key = str(
        item.get("originalSourceKey") or ""
    ).strip()

    if not source_key:
        return cleanup_status

    source_version_id = normalize_optional_version_id(
        item.get("originalSourceVersionId")
    )

    return cleanup_source_after_move(
        document_id=item["documentId"],
        source_key=source_key,
        source_version_id=source_version_id,
    )


def update_cleanup_status(
    document_id: str,
    status: str,
    error_message: str | None = None,
) -> None:
    now = utc_now()
    expression_values: dict[str, Any] = {
        ":cleanupStatus": status,
        ":cleanupAt": now,
        ":updatedAt": now,
    }
    update_expression = """
        SET sourceCleanupStatus = :cleanupStatus,
            sourceCleanupAt = :cleanupAt,
            updatedAt = :updatedAt
    """

    if error_message:
        update_expression += (
            ", sourceCleanupError = :cleanupError"
        )
        expression_values[
            ":cleanupError"
        ] = error_message[:1000]

    table.update_item(
        Key={"documentId": document_id},
        UpdateExpression=update_expression,
        ExpressionAttributeValues=expression_values,
    )


def get_allowed_source_statuses(
    decision_source: str,
) -> set[str]:
    if decision_source == "ADMIN_REVIEW":
        return set(ADMIN_REVIEW_STATUSES)

    return set(SYSTEM_SOURCE_STATUSES)


def is_already_completed(
    item: dict,
    status: str,
    prefix: str,
    decision: str,
) -> bool:
    return (
        normalized_upper(item.get("status")) == status
        and str(item.get("currentPrefix") or "").strip() == prefix
        and normalized_upper(item.get("finalDecision")) == decision
    )


# =========================================================
# Utilities
# =========================================================

def normalized_upper(
    value: Any,
    default: str = "",
) -> str:
    return str(value if value is not None else default).strip().upper()


def normalize_optional_version_id(value: Any) -> str | None:
    if value is None:
        return None

    normalized = str(value).strip()
    if normalized.upper() in {
        "",
        "NONE",
        "NULL",
        "UNKNOWN",
    }:
        return None

    return normalized


def decimal_value(value: Any) -> Decimal:
    if isinstance(value, Decimal):
        return value

    try:
        return Decimal(str(value if value is not None else 0))
    except (InvalidOperation, ValueError, TypeError):
        return Decimal("0")


def utc_now() -> str:
    return datetime.now(timezone.utc).isoformat()


def api_response(
    status_code: int,
    body: dict,
) -> dict:
    return {
        "statusCode": status_code,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": CORS_ORIGIN,
        },
        "body": json.dumps(body, default=str),
    }

def validate_internal_admin_review(event):
    internal_invocation = event.get(
        "internalInvocation"
    )

    reviewed_by = str(
        event.get("reviewedBy", "")
    ).strip()

    review_id = str(
        event.get("reviewId", "")
    ).strip()

    if internal_invocation is not True:
        raise PermissionError(
            "ADMIN_REVIEW chỉ được gọi nội bộ"
        )

    if not reviewed_by:
        raise PermissionError(
            "Thiếu reviewedBy"
        )

    if not review_id:
        raise PermissionError(
            "Thiếu reviewId"
        )

    print(json.dumps({
        "event": "INTERNAL_ADMIN_REVIEW_AUTHORIZED",
        "reviewedBy": reviewed_by,
        "reviewId": review_id
    }))
```

Trong tab **Configuration**, chọn **Environment variables** và thêm:

| Key | Value |
| --- | --- |
| `CLEAN_PREFIX `| `clean `|
| `DOCUMENTS_TABLE` | `SecureDocuments` |
| `DOCUMENT_BUCKET` | `secure-ai-document-storage` |
| `QUARANTINE_PREFIX` | `quarantine` |
| `REJECT_PREFIX` | `reject` |

![Environment variables](/images/5-Workshop/5.7/5.7.1/5.7.1.2.png)

Trong phần **General configuration**, cấu hình:

+ **Memory**: `512 MB`
+ **Timeout**: `1 phút`

![General configuration](/images/5-Workshop/5.8-Decision/5.8.1-create-decision-lambda/general-configuration.png)

Role thực thi của Lambda cần có các quyền chính:

+ `s3:GetObject`
+ `s3:HeadObject`
+ `s3:PutObject`
+ `s3:DeleteObject`
+ `dynamodb:GetItem`
+ `dynamodb:UpdateItem`
+ Quyền ghi log vào Amazon CloudWatch Logs.

```text
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "ReadAndUpdateSecureDocuments",
			"Effect": "Allow",
			"Action": [
				"dynamodb:GetItem",
				"dynamodb:UpdateItem"
			],
			"Resource": "arn:aws:dynamodb:ap-southeast-1:accout_id:table/SecureDocuments"
		},
		{
			"Sid": "ReadCleanScannedQuarantineObjects",
			"Effect": "Allow",
			"Action": [
				"s3:*"
			],
			"Resource": "arn:aws:s3:::secure-ai-document-storage/quarantine/*"
		},
		{
			"Sid": "WriteCleanAndRejectObjects",
			"Effect": "Allow",
			"Action": [
				"s3:*"
			],
			"Resource": [
				"arn:aws:s3:::secure-ai-document-storage/clean/*",
				"arn:aws:s3:::secure-ai-document-storage/reject/*"
			]
		},
		{
			"Sid": "DeleteQuarantineObjects",
			"Effect": "Allow",
			"Action": [
				"s3:*"
			],
			"Resource": "arn:aws:s3:::secure-ai-document-storage/quarantine/*"
		}
	]
}
```
**do một phần triền khai bị lỗi nên cấp full quyền quarantine**
Phạm vi quyền S3 nên giới hạn trong bucket:

```text
secure-ai-document-storage
```

và các prefix:

```text
quarantine/*
clean/*
reject/*
```

![Lambda permissions](/images/5-Workshop/5.7/5.7.1/5.7.1.4.png)

Lambda hỗ trợ hai nguồn quyết định:

+ `SYSTEM_RULES`: quyết định tự động từ kết quả GuardDuty và AI.
+ `ADMIN_REVIEW`: quyết định thủ công do quản trị viên gửi từ API đánh giá tài liệu.

{{% notice warning %}}
Giá trị `ADMIN_GROUP` phải giống chính xác tên nhóm trong Amazon Cognito. Nếu nhóm đã tạo có tên `Admins`, không cấu hình giá trị thành `admin`.
{{% /notice %}}

{{% notice note %}}
Decision Engine chỉ cho phép chuyển tài liệu sang vùng `clean/` khi GuardDuty đã trả về `NO_THREATS_FOUND`.
{{% /notice %}}