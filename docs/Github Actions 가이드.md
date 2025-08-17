📑 Github Test Suite 에러/경고 정리 가이드 (Plan\_v1.0)

본 문서는 **Plan\_v1.0 (Next.js + TypeScript + Tailwind)** 프로젝트 개발 중 Github Actions CI/CD 상에서 발생한 주요 **lint/type/test suite 에러**들을 정리하고, 재발 방지를 위한 기준을 제시합니다.

---

## 1. Context 관련 에러

### 문제

* `TaskContext`, `DataContext`가 로컬 선언만 되어 있고 `export` 누락.
* `App.tsx`에서 존재하지 않는 `AuthProvider`를 import.

### 해결 기준

* **모든 Context는 `Context`, `Provider`, `Hook` 3세트를 export**.
* `contexts/index.ts` 배럴에서 실제 존재하는 Provider만 export.
* `App.tsx`에서는 존재하는 Provider만 감싸서 사용.

---

## 2. groupId 관련 에러

### 문제

* `groupId`라는 이름이 정의되지 않았는데 참조 (`useActivity.ts`, `DataContext.tsx` 등).
* Jest test suite에서 `Cannot find name 'groupId'` 발생.

### 해결 기준

* `_groupId` 또는 `group?.id` 형태로 안전하게 접근.
* 재사용 패턴:

  ```ts
  const gid = typeof _groupId !== 'undefined' && _groupId !== null
    ? _groupId
    : (group?.id ?? (group as any)?.groupId);
  ```

---

## 3. 타입 불일치 에러

### ProfileSection.tsx

* `UserProfile`을 `Record<string, unknown>` 파라미터에 그대로 전달.
* **해결**: `Record<string, unknown>`으로 변환 후 전달.

  ```ts
  const payload: Record<string, unknown> = { ...userProfile };
  updateProfile(payload);
  ```

### DataSection.tsx

* `Record<string, unknown>[]`을 상태 `Backup[]`에 그대로 set.
* **해결**: 매핑 변환 후 set.

  ```ts
  const toBackup = (r: Record<string, unknown>): Backup => ({
    name: String(r.name ?? ''),
    timestamp: Number(r.timestamp ?? Date.now()),
    frequency: String(r.frequency ?? 'manual'),
    path: String(r.path ?? ''),
  });
  setBackups((prev) => [...prev, ...records.map(toBackup)]);
  ```

---

## 4. ESLint 관련 에러

### 문제

* `error`, `err` 선언 후 미사용.
* `any` 타입 다수.
* `useEffect` 내부 함수 재선언으로 deps 경고.

### 해결 기준

* 미사용 변수는 `_error`, `_err` 접두어 처리.
* `any` → 구체 타입/인터페이스로 교체.
* `useEffect` 내부 함수는 `useCallback`으로 감싸거나 deps 주석 처리.

---

## 5. 파일별 주요 정리

* **lib/notifications.ts**

  * any 제거 → `PushPayload` 인터페이스 정의.
  * catch 블록은 `_error` 사용.

* **lib/imageUtils.ts**

  * 미사용 import(`getDownloadURL`, `ref`, `uploadBytes`, `storage`) 제거.
  * catch 블록 `_error` 처리.

* **lib/firestore.ts / points.ts / pointsAnalyzer.ts / statisticsAnalyzer.ts / storage.ts**

  * `try {}` 뒤에 `catch`/`finally` 누락 → 보강.

* **lib/realtime.ts**

  * `onSnapshot(q, , handler)` 같은 빈 인자 → 정상 시그니처(`onSnapshot(q, snapHandler, errorHandler)`)로 교정.

---

## 6. 재발 방지 체크리스트

* [ ] Context 추가 시 항상 export 여부 확인 (Context/Provider/Hook).
* [ ] groupId 사용 시 `_groupId` 또는 `group?.id` 안전 접근 패턴 활용.
* [ ] 외부 호출 함수 타입은 반드시 맞춰 변환 후 전달.
* [ ] ESLint 규칙(@typescript-eslint/no-unused-vars, no-explicit-any 등) 준수.
* [ ] try/catch/finally 블록 완전성 확인.
* [ ] Github Actions test suite를 로컬에서도 실행하여 CI 실패 사전 방지.

---

✅ 이 가이드는 **Plan\_v1.0** 개발 전 과정에서 참조해야 하는 품질 보증 기준이며, 모든 신규 PR은 이 기준에 맞춰 리뷰/검증되어야 합니다.
