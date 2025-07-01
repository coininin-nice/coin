# User gRPC Service

## 내가 짠 코드는 디코로 보내드림 ( 이건 AI 딸깍 정리라 좀 별로임 )

사용자 정보 조회를 위한 gRPC 서비스 구현입니다.

## 📋 개요

내부 시스템 간 사용자 정보를 조회하기 위한 gRPC 서비스로, UUID를 통해 사용자 정보를 반환합니다.

## 🏗️ 아키텍처

```
Client → gRPC Service → Use Case → Domain
```

## 📁 프로젝트 구조

```
user/
├── infrastructure/grpc/server/
│   ├── dto/
│   │   └── InternalUserResponse.kt
│   ├── mapper/
│   │   └── UserGrpcMapper.kt
│   └── UserGrpcService.kt
└── proto/
    └── user_service.proto
```

## 🔧 주요 컴포넌트

### 1. Proto Definition (`user_service.proto`)

```protobuf
syntax = "proto3";
package casper.user;

service UserService {
  rpc GetUserInfoByUserId(GetUserInfoRequest) returns (GetUserInfoResponse);
}

message GetUserInfoRequest {
  string user_id = 1;
}

message GetUserInfoResponse {
  string id = 1;
  string phone_number = 2;
  string name = 3;
  bool is_parent = 4;
  UserRole role = 5;
}

enum UserRole {
  UNSPECIFIED = 0;
  ROOT = 1;
  USER = 2;
  ADMIN = 3;
}
```

### 2. DTO (`InternalUserResponse.kt`)

내부 시스템 간 사용자 정보 응답 데이터를 담는 DTO 클래스입니다.

```kotlin
data class InternalUserResponse(
    val id: UUID,
    val phoneNumber: String,
    val name: String,
    val isParent: Boolean,
    val receiptCode: Long?,
    val role: UserRole,
)
```

### 3. Mapper (`UserGrpcMapper.kt`)

gRPC 통신을 위한 변환 작업을 담당하는 매퍼 클래스입니다.

```kotlin
@Component
class UserGrpcMapper {
    fun toGetUserInfoResponse(userResponse: InternalUserResponse): UserServiceProto.GetUserInfoResponse
    private fun toProtoUserRole(userRole: UserRole): UserServiceProto.UserRole
}
```

**주요 기능:**
- `InternalUserResponse` → `GetUserInfoResponse` 변환
- `UserRole` enum 매핑 (Domain ↔ Proto)

### 4. gRPC Service (`UserGrpcService.kt`)

사용자 관련 gRPC 서비스 구현체입니다.

```kotlin
@GrpcService
class UserGrpcService(
    private val queryUserByUUIDUseCase: QueryUserByUUIDUseCase,
    private val userGrpcMapper: UserGrpcMapper,
) : UserServiceGrpcKt.UserServiceCoroutineImplBase()
```

## 🔄 API 명세

### GetUserInfoByUserId

사용자 UUID를 통해 사용자 정보를 조회합니다.

**Request:**
```protobuf
message GetUserInfoRequest {
  string user_id = 1;  // UUID 형식
}
```

**Response:**
```protobuf
message GetUserInfoResponse {
  string id = 1;           // 사용자 UUID
  string phone_number = 2; // 전화번호
  string name = 3;         // 사용자 이름
  bool is_parent = 4;      // 학부모 여부
  UserRole role = 5;       // 사용자 권한
}
```

**UserRole Enum:**
- `UNSPECIFIED = 0`: 미지정
- `ROOT = 1`: 최고 관리자
- `USER = 2`: 일반 사용자  
- `ADMIN = 3`: 관리자

## ⚠️ 예외 처리

| 예외 상황 | gRPC Status | 설명 |
|----------|-------------|------|
| 잘못된 UUID 형식 | `INVALID_ARGUMENT` | Invalid UUID format |
| 서버 내부 오류 | `INTERNAL` | 서버 오류 |

## 🛠️ 기술 스택

- **Language**: Kotlin
- **Framework**: Spring Boot
- **gRPC**: grpc-kotlin
- **Build Tool**: Gradle
- **Libraries**: 
  - `net.devh.boot.grpc.server` - Spring Boot gRPC integration

## 📝 사용 예제

```kotlin
// gRPC 클라이언트에서 호출
val request = GetUserInfoRequest.newBuilder()
    .setUserId("550e8400-e29b-41d4-a716-446655440000")
    .build()

val response = userServiceStub.getUserInfoByUserId(request)
```

## 🔍 특징

- **코루틴 기반**: `UserServiceCoroutineImplBase` 상속으로 비동기 처리
- **클린 아키텍처**: Use Case 패턴을 통한 비즈니스 로직 분리
- **타입 안전성**: Kotlin과 Proto의 강타입 시스템 활용
- **예외 안전성**: gRPC Status를 통한 명확한 에러 처리
