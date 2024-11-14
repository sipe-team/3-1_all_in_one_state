# React 19에서 상태의 개념과 동작원리가 변하는가? (심미진)

우선 액션(Actions)에 대해서 알아보면 Action은 **컨벤션에 따라 비동기 트랜지션을 사용하는 함수**를 의미한다.

React19에서는 트랜지션에서 비동기 함수를 사용하여 대기 상태, 에러, 양식, 낙관적 업데이트를 자동으로 처리하는 기능이 추가되었다.

<br>

## useTransition

그 중 `useTransition`을 사용하여 대기 상태를 다룬다.

```jsx
// 액션에서 대기 상태 사용하기
function exampleTransition() {
  const [selectedId, setSelectedId] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleImageSelect = async (id) => {
    startTransition(async () => {
      // React 19에서는 비동기 함수를 직접 startTransition에 전달 가능
      const imageData = await fetchImageDetails(id);
      setSelectedId(imageData);
    });
  };

  return (
    <div>
      <div className="gallery-grid">
        {images.map((image) => (
          <button
            key={image.id}
            onClick={() => handleImageSelect(image.id)}
            disabled={isPending}
          >
            <img src={image.thumbnail} alt={image.title} />
          </button>
        ))}
      </div>

      {isPending ? (
        <div>Loading image details...</div>
      ) : (
        selectedId && <ImageDetails id={selectedId} />
      )}
    </div>
  );
}
```

비동기 트랜지션은 `isPending` 값을 즉시 true로 설정하며 비동기 요청한다.

그 후 트랜지션이 수행되면 `isPending`을 false로 전환한다.

이러한 방식은 데이터가 변경되는 동안에도 현재 UI의 반응성 및 상호작용을 유지 가능하다.

<br>

## useActionState

React19는 액션의 일반적인 사용을 다루기 위한 새로운 훅 [React.useActionstate](https://react.dev/blog/2024/04/25/react-19#new-hook-useactionstate)을 도입해 단순하게 작성 가능하다

```jsx
// <form> 액션과 useActionState 사용
function ChangeName({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

`useActionState`는 함수(action 함수)를 받아 호출할 래핑된 액션을 반환한다.

이는 각 액션이 조합되므로 가능하다.

래핑된 액션이 호출되면 `useActionState`는 액션의 마지막 결과를 `data`로, 액션의 대기 상태를 `pending`으로 반환한다.

<br>

## useFormStatus

컴포넌트가 속한 `<form>`에 대한 정보를 접근을 좀 더 쉽게 다루도록 `useFormStatus` 훅을 새롭게 추가했다.

```jsx
import { useFormStatus } from "react-dom";

function DesignButton() {
  const { pending } = useFormStatus();
  return <button type="submit" disabled={pending} />;
}
```

`useFormStatus` 훅은 context provider처럼 부모의 `<form>`의 상태를 읽는다.

<br>

## useOptimistic

데이터 변경을 수행할 때 비동기 요청이 진행되는 동안 최종 상태를 낙관적으로 표시하는 것도 흔한 UI 패턴

```jsx
function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    likes,
    (currentLikes) => currentLikes + 1
  );

  const handleLike = async () => {
    addOptimisticLike();
    try {
      const updatedLikes = await likePost(postId);
      setLikes(updatedLikes);
    } catch (error) {
      // 에러 발생 시 optimisticLikes는 자동으로 원래 상태로 롤백됨
      console.error("Failed to like post:", error);
    }
  };

  return (
    <button onClick={handleLike} className="like-button">
      ❤️ {optimisticLikes} likes
    </button>
  );
}
```

useOptimistic은 비동기 작업의 결과를 기다리는 동안 즉각적인 UI 피드백을 제공한다.

`const [optimisticValue, addOptimisticUpdate] = useOptimistic(value, updateFn)`

- value: 실제 상태 값
- updateFn: 낙관적 업데이트를 수행할 함수

<br>

## use

Promise, Context 등의 값을 동기적으로 읽을 수 있게 해주는 use 훅을 도입했다.

Suspense와 함께 작동하여 데이터 로딩 상태를 처리하고 컴포넌트 내 어디서나 사용 가능한 특징을 가진다. (조건문, 반복문 포함)

```javascript
function UserProfile({ userId }) {
  const user = use(fetchUser(userId));
  const posts = use(fetchUserPosts(userId));

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <h2>Posts</h2>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```
