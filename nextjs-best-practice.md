# Next.js Best Practice

参考:
- https://zenn.dev/akfm/books/nextjs-basic-principle
- https://youtu.be/Ca1h3KUfQ5k?si=yvnlllW-Vb2UcCj1

## cursor rules

by 生成AI
```plain text
# Next.js App Router Best Practices

## データフェッチ
- Server Componentsでデータフェッチを実行し、クライアントサイドでの不要なAPI呼び出しを避ける
- データフェッチは使用するコンポーネントと同じ場所（コロケーション）に配置する
- Request Memoizationを活用して重複するリクエストを防ぐ
- 並行データフェッチを使用してパフォーマンスを向上させる
- N+1問題を避けるため、DataLoaderパターンや適切なAPI設計を採用する
- 細粒度のREST API設計を行い、必要なデータのみを取得する

## コンポーネント設計
- デフォルトでServer Componentsを使用し、必要な場合のみClient Componentsに変更する
- Client Componentsは以下の場合に使用する：
  - イベントリスナーが必要な場合
  - useState、useEffectなどのReactフックを使用する場合
  - ブラウザ専用APIを使用する場合
- useEffectの多用を避け、Server Componentsでのデータフェッチを優先する
- Compositionパターンを活用してコンポーネントの再利用性を高める
- Container/Presentationalパターンで関心事を分離する
- Container 1stな設計を採用し、ディレクトリ構成を整理する

## キャッシュ戦略
- Static RenderingとFull Route Cacheを活用してパフォーマンスを向上させる
- Dynamic RenderingとData Cacheを適切に組み合わせる
- Router Cacheを理解し、適切に活用する
- データ操作にはServer Actionsを使用する（イベントハンドラではなく）
- 必要に応じてDynamic IOを検討する

## レンダリング最適化
- Server Componentsの純粋性を保つ
- SuspenseとStreamingを活用してユーザー体験を向上させる
- ストリーミングデータフェッチングでスケルトン表示を実装する
- Partial Pre Rendering (PPR)を検討する（Experimental）

## ルーティングとナビゲーション
- ルートグループ（括弧付きディレクトリ）はルーティングとして認識されない
- useSearchParamsや動的ルーティングのパラメータは非同期で受け取る
- 動的ルーティング（/blog/[id]）ではasync/awaitを使用する

## データベース・API統合
- Supabaseを使用する際は適切なクライアントを選択する：
  - クライアント側: createClient() (@supabase/supabase-js)
  - サーバー側: createServerClient() (@supabase/ssr)

## よくある間違いと対処法

### ① ルートグループの誤解
**問題**: `/(admin)/page.tsx`などのルートグループをネストルーティングとして認識してしまう
**解決策**: ルートグループはルーティングとして認識されません。トップページの`page.tsx`と競合するのでエラーになります。

```tsx
// ❌ 間違い
app/
  (admin)/
    page.tsx  // これは /page.tsx と同じルートになる

// ✅ 正しい
app/
  admin/
    page.tsx  // これは /admin/page.tsx になる
```

### ② useEffectの多用
**問題**: useEffect()を多用してしまう
**解決策**: ベストプラクティスは「サーバーコンポーネントでのデータフェッチ」です。DAL等でデータフェッチ用のファイルを作成しておき、呼び出すときはサーバーコンポーネントで行うようにしましょう。

```tsx
// ❌ よくある間違い
'use client';
export default function Page() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return <div>{data}</div>;
}

// ✅ 正しい実装
export default async function Page() {
  const data = await fetchData(); // サーバーコンポーネントで直接await
  return <div>{data}</div>;
}
```

### ③ ストリーミングデータフェッチングの未活用
**問題**: ストリーミングデータフェッチングしてくれない
**解決策**: サーバーコンポーネントの利用に加えて、各コンポーネントでデータ取得時にスケルトン表示したい場合は「ストリーミングデータフェッチング」をSuspenseで行うように指示しましょう。

```tsx
// ✅ ストリーミングデータフェッチングの実装例
import { Suspense } from 'react';

// スケルトンコンポーネント
function Skeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-6 bg-gray-200 rounded mb-2"></div>
      <div className="h-4 bg-gray-200 rounded mb-1"></div>
      <div className="h-4 bg-gray-200 rounded w-5/6"></div>
    </div>
  );
}

// データフェッチ用のコンポーネント
async function DataComponent() {
  const data = await fetchData();
  return <div>{/* データ表示 */}</div>;
}

// メインページ
export default function Page() {
  return (
    <div>
      <h1>ページタイトル</h1>
      <Suspense fallback={<Skeleton />}>
        <DataComponent />
      </Suspense>
    </div>
  );
}
```

### ④ Server Actionsの未使用
**問題**: Server Actionsを使ってくれない
**解決策**: 「Server Actionsで実装して」と言わないとイベントハンドラで実装するのがデフォルトになっている。できるならServer Actionsが理想です。

```tsx
// ✅ Server Actionsの実装例
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title');
  const content = formData.get('content');
  
  await db.posts.create({ title, content });
  revalidateTag('posts');
}

// フォームコンポーネント
export default function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="タイトル" />
      <textarea name="content" placeholder="内容" />
      <button type="submit">投稿</button>
    </form>
  );
}
```

### ⑤ 非同期パラメータ処理の忘れ
**問題**: useSearchParamsや動的ルーティングの`/blog/[id]`等のparams受け取る際は、非同期で受け取る
**解決策**: async/awaitで受け取らないとエラーになります。

```tsx
// ✅ 動的ルーティングでの非同期処理
export default async function BlogPost({ params }: { params: { id: string } }) {
  const post = await getPost(params.id);
  return <article>{post.content}</article>;
}

// ✅ useSearchParamsでの非同期処理
'use client';
import { useSearchParams } from 'next/navigation';

export default function SearchResults() {
  const searchParams = useSearchParams();
  const query = searchParams.get('q');
  
  return <div>検索結果: {query}</div>;
}
```

### ⑥ Supabaseクライアントの使い分け（Supabaseを使う場合は）
**問題**: Supabaseをサーバー側で実行する場合はcreateServerClient()を利用する
**解決策**: クライアント側で実行する場合はcreateClient()で、サーバー側で実行する場合はcreateServerClient()を利用するのが正解です。モジュールは@supabase/supabase-js @supabase/ssrを利用しています。

```tsx
// ✅ クライアント側での使用
'use client';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

// ✅ サーバー側での使用
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export default async function ServerComponent() {
  const cookieStore = cookies();
  
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value;
        },
      },
    }
  );
  
  const { data } = await supabase.from('posts').select('*');
  return <div>{/* データ表示 */}</div>;
}
```

## その他のプラクティス
- リクエストの参照とレスポンスの操作を適切に行う
- 認証と認可を適切に実装する
- エラーハンドリングを包括的に行う
- 適切なエラーバウンダリを設定する

## パフォーマンス最適化
- 画像の最適化（next/image）を活用する
- フォントの最適化（next/font）を行う
- コード分割を適切に行う
- バンドルサイズを監視し、最適化する

## セキュリティ
- 環境変数を適切に管理する
- 入力値の検証を行う
- CSRF対策を実装する
- 適切なCORS設定を行う
