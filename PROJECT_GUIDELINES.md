# Project Guidelines

이 문서는 본 프로젝트의 아키텍처, 폴더 구조, 개발 규칙, 기술 스택, 코드 품질 및 협업 기준을 명확히 안내합니다. 모든 개발자는 아래 가이드라인을 반드시 준수해야 합니다.

---

## Architecture & Project Structure

- **Clean Architecture + Feature-First** 구조를 기본 패턴으로 사용합니다.
- 코드는 세 개의 주요 계층(data, domain, presentation)으로 나눕니다.
- 각 기능(Feature)은 `lib/features/` 하위에 별도 모듈로 생성합니다. (예: `lib/features/auth`, `lib/features/home`)
- 공통 유틸리티는 `lib/core/`, 재사용 UI 컴포넌트는 `lib/shared/`에 둡니다.
- **의존성 규칙:** 바깥 계층(예: presentation)은 안쪽 계층(domain, data)에만 의존할 수 있으며, 반대는 허용하지 않습니다.

```
lib/
  app/                # 앱 진입점, 라우팅 등
  core/               # 전역 상수, 서비스, 유틸리티
  features/           # 도메인(기능)별 모듈 (data/domain/presentation 계층 분리)
  shared/             # 여러 feature에서 재사용되는 위젯, extension 등
  main.dart           # 앱 실행 진입점
```

---

## State Management

- **Riverpod 2.x**만을 상태 관리 솔루션으로 사용합니다.
- 복잡한 상태(API 호출, side effect)는 `AsyncNotifierProvider` 사용
- 단순 로컬 상태는 `NotifierProvider` 사용
- 1회성 비동기 작업은 `FutureProvider` 사용
- 실시간 데이터 스트림은 `StreamProvider` 사용
- 항상 `riverpod_generator`를 통한 코드 생성으로 타입 안정성 확보

---

## Networking & Data

- **Dio**를 기본 HTTP 클라이언트로 사용
- **Retrofit** + 코드 생성으로 API 서비스 정의
- 데이터 소스 추상화를 위해 **Repository 패턴** 적용
- **Freezed**로 불변 데이터 클래스 및 JSON 직렬화 구현
- 에러 처리는 커스텀 Exception/Failure 클래스로 일관성 있게 처리
- 필요시 **offline-first** 전략 적용

---

## UI/UX & Theming

- **Material 3 (Material You)**를 기본 디자인 시스템으로 사용
- 라이트/다크 테마 모두 지원
- 모바일/태블릿/데스크탑 등 다양한 화면 크기에 대응하는 반응형 레이아웃 구현
- **flutter_hooks**로 위젯 생명주기 및 로컬 상태 관리
- 접근성(semantic label, 명암비 등) 항상 고려

---

## Navigation & Routing

- **GoRouter**만을 라우팅 솔루션으로 사용
- 선언적 라우팅 및 route definition 적용
- 타입 안전한 네비게이션(생성된 route parameter 활용)
- 딥링크 및 네비게이션 상태 복원 지원
- 인증 등 라우트 가드 구현 필수

---

## Component Structure

- feature별 위젯은 해당 feature의 `presentation/widgets/`에 위치
- 재사용 UI 컴포넌트는 `lib/shared/widgets/`에 위치
- 위젯은 복잡도에 따라 atoms(기본), molecules(복합), organisms(복잡) 등으로 분류
- 모든 커스텀 위젯은 StatelessWidget, StatefulWidget, HookWidget을 상속
- const 생성자 적극 사용(성능 최적화)

---

## Animation & Interactions

- Flutter 내장 애니메이션 API 우선 사용
- 복잡한 애니메이션은 Rive/Lottie 등 외부 도구 고려
- GoRouter의 transition builder로 부드러운 페이지 전환 구현
- 간단한 애니메이션은 AnimatedContainer, AnimatedOpacity, AnimatedPositioned 등 활용
- Material Design 모션 가이드라인 준수

---

## Development Tools & Code Quality

- **Very Good Analysis**로 Dart/Flutter 린트 규칙 적용
- 단위/위젯/통합 테스트를 충분히 작성
- 코드 생성 작업은 build_runner 사용
- 커밋 전 `dart format`으로 코드 포맷팅
- `flutter analyze`로 잠재적 이슈 사전 탐지

---

## Package Management

- 공식 Flutter 팀 패키지 우선 사용
- pub.dev 평점이 높은 커뮤니티 패키지 사용
- 비표준 패키지 사용 시 사유를 문서화
- pubspec.yaml에 버전 고정하여 재현성 확보
- 정기적으로 의존성 업데이트 및 호환성 테스트

---

## Local Storage & Persistence

- 간단한 키-값 저장은 SharedPreferences 사용
- 복잡한 로컬 DB는 Hive 또는 Isar 사용
- 민감 정보는 flutter_secure_storage로 안전하게 저장
- 네트워크 응답 캐시는 만료 전략 포함하여 적절히 관리
- 스토리지 실패 시 graceful fallback 구현

---

## Internationalization & Localization

- Flutter 내장 Intl 패키지로 i18n 지원
- 모든 사용자 노출 문자열은 로컬라이제이션 파일로 분리
- 필요시 RTL 언어 지원
- 날짜/숫자/통화 등은 각 로케일에 맞게 포맷팅
- 다양한 로케일로 앱을 테스트

---

## Example Code Templates

아래는 각 계층별(모델, 레포지토리, 데이터소스, 엔티티, 유스케이스, Provider, 위젯 등) 예시 코드입니다. 필요할 때 복사해서 사용하세요.

### Data Layer - Model (Freezed)
```dart
// user_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';
part 'user_model.freezed.dart';
part 'user_model.g.dart';

@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String email,
    String? name,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) => _$UserModelFromJson(json);
}
```

### Data Layer - Repository Implementation
```dart
// auth_repository_impl.dart
import '../../domain/repositories/auth_repository.dart';
import '../datasources/auth_remote_datasource.dart';
import '../models/user_model.dart';

class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  AuthRepositoryImpl(this.remoteDataSource);

  @override
  Future<UserModel?> login(String email, String password) async {
    return await remoteDataSource.login(email, password);
  }
}
```

### Data Layer - Remote DataSource (Dio/Retrofit)
```dart
// auth_remote_datasource.dart
import '../models/user_model.dart';

abstract class AuthRemoteDataSource {
  Future<UserModel?> login(String email, String password);
}
// 실제 구현은 Retrofit을 활용하여 생성
```

### Domain Layer - Entity
```dart
// user.dart
class User {
  final String id;
  final String email;
  final String? name;

  const User({required this.id, required this.email, this.name});
}
```

### Domain Layer - Repository Interface
```dart
// auth_repository.dart
import '../entities/user.dart';

abstract class AuthRepository {
  Future<User?> login(String email, String password);
}
```

### Domain Layer - UseCase
```dart
// login_usecase.dart
import '../repositories/auth_repository.dart';
import '../entities/user.dart';

class LoginUseCase {
  final AuthRepository repository;
  LoginUseCase(this.repository);

  Future<User?> call(String email, String password) async {
    return await repository.login(email, password);
  }
}
```

### Presentation Layer - Provider (Riverpod)
```dart
// login_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

final loginProvider = AsyncNotifierProvider<LoginNotifier, void>(LoginNotifier.new);

class LoginNotifier extends AsyncNotifier<void> {
  @override
  Future<void> build() async {
    // 초기화 로직
  }

  Future<void> login(String email, String password) async {
    state = const AsyncLoading();
    // await ...
    state = const AsyncData(null);
  }
}
```

### Presentation Layer - Page
```dart
// login_page.dart
import 'package:flutter/material.dart';
import '../providers/login_provider.dart';

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('로그인')),
      body: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              decoration: const InputDecoration(labelText: '이메일'),
            ),
            const SizedBox(height: 16),
            TextField(
              decoration: const InputDecoration(labelText: '비밀번호'),
              obscureText: true,
            ),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () {
                // Provider 연동 예시
                // context.read(loginProvider.notifier).login(email, password);
              },
              child: const Text('로그인'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Presentation Layer - Widget
```dart
// custom_button.dart
import 'package:flutter/material.dart';

class CustomButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  const CustomButton({super.key, required this.label, required this.onPressed});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.indigo,
        foregroundColor: Colors.white,
        padding: const EdgeInsets.symmetric(vertical: 16, horizontal: 24),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
      onPressed: onPressed,
      child: Text(label, style: const TextStyle(fontSize: 16, fontWeight: FontWeight.w600)),
    );
  }
}
```

### Core Layer - API Service
```dart
// api_service.dart
import 'package:dio/dio.dart';

class APIService {
  final Dio dio;
  APIService(this.dio);

  Future<Response> get(String path) async {
    return await dio.get(path);
  }
}
```

### Core Layer - Storage Service
```dart
// storage_service.dart
import 'package:shared_preferences/shared_preferences.dart';

class StorageService {
  Future<void> saveToken(String token) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('token', token);
  }

  Future<String?> getToken() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString('token');
  }
}
```

---

> **이 가이드라인에 맞지 않는 구조/코드는 반드시 수정해야 하며, 예외가 필요한 경우 팀 내 논의 후 문서화해야 합니다.** 