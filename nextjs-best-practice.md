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
- Compositionパターンを活用してコンポーネントの再利用性を高める
- Container/Presentationalパターンで関心事を分離する
- Container 1stな設計を採用し、ディレクトリ構成を整理する

## キャッシュ戦略
- Static RenderingとFull Route Cacheを活用してパフォーマンスを向上させる
- Dynamic RenderingとData Cacheを適切に組み合わせる
- Router Cacheを理解し、適切に活用する
- データ操作にはServer Actionsを使用する
- 必要に応じてDynamic IOを検討する

## レンダリング最適化
- Server Componentsの純粋性を保つ
- SuspenseとStreamingを活用してユーザー体験を向上させる
- Partial Pre Rendering (PPR)を検討する（Experimental）

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

## 開発体験
- TypeScriptを活用する
- ESLintとPrettierを設定する
- 適切なテスト戦略を立てる
- 開発環境とプロダクション環境の違いを理解する
