# TanStack Query 심층 분석: 아키텍처와 활용 전략

## 1. 데이터 로딩 전략

TanStack Query는 두 가지 핵심적인 데이터 로딩 전략을 제공

### Refetching
- 데이터 자동 갱신을 위한 전략
- 시간 기반 캐시 관리를 통해 데이터 신선도 유지
- 단일 데이터 포인트 관리에 적합

### Infinite Query
- 페이지 기반의 데이터 관리 전략
- 데이터를 누적하며 점진적으로 로드
- 무한 스크롤과 같은 UI 패턴에 적합
- 양방향 페이지네이션 지원

## 2. 캐시 관리와 무효화 전략

### 캐시 라이프사이클
- `staleTime`: 데이터를 fresh 상태로 유지하는 시간 (기본값: 0)
- `gcTime`: 비활성 쿼리의 캐시 유지 시간 (기본값: 5분)
- Suspense Query의 경우 staleTime 기본값은 1초로 설정

### 캐시 무효화 방법
1. 자동 refetch
   - 윈도우 포커스
   - 컴포넌트 마운트
   - 주기적 갱신

2. 수동 무효화
   - `queryClient.invalidateQueries()` 사용
   - 특정 쿼리 키 기반 무효화

3. 낙관적 업데이트
   - UI 먼저 업데이트 후 서버 응답 대기
   - 실패 시 롤백 가능

## 3. Query Key 설계 전략

### 효과적인 키 구성
- 일반적인 키에서 구체적인 키 순으로 구성
- 객체 내부 키 순서는 중요하지 않음
- useQuery와 useInfiniteQuery는 동일 키 사용 불가

### 쿼리 키 팩토리 패턴
```js
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const
}
```

## 4. Next.js와의 통합

### 구현 방식
1. `prefetch + de/hydrate`
   - 서버에서 데이터 미리 로드
   - HTML에 데이터 포함

2. Streaming 사용
   - React 18의 Suspense 활용
   - 점진적 UI 렌더링

3. 실험적 API 사용
   - `@tanstack/react-query-next-experimental` 활용
   - 단순화된 구현 가능

### 고려사항
- 서버 컴포넌트와의 호환성
- 클라이언트/서버 컴포넌트 분리 필요성
- 인증 상태 관리와의 통합

## 5. 사용 시 고려사항

### React Query가 필요한 경우
- 낙관적 업데이트와 롤백 필요
- 실시간 폴링 구현
- 복잡한 로딩/에러 처리
- 무한 스크롤 구현
- 오프라인 지원 필요

### Next.js의 fetch만으로 충분한 경우
- 정적 페이지 중심
- 서버 사이드 렌더링 위주
- 실시간 업데이트가 적은 경우

