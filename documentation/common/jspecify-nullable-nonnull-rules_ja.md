# JSpecify @Nullable / @NonNull 運用ルール

## 大前提

- 全パッケージの `package-info.java` に `@NullMarked` を設定する。
- `@NullMarked` 環境では、アノテーションなしの型は `@NonNull` として扱われる。
- 例外: Java アノテーション定義クラスのパッケージには `@NullUnmarked` を使用する
  （アノテーション型には null 解析が不要なため）。

---

## 1. `@Nullable` を付ける箇所

### パラメータ

| パターン | 例 | 理由 |
| --- | --- | --- |
| `Locale` パラメータ | `@Nullable Locale locale` | `null` の場合は `Locale.ROOT` で代替（バッチ処理等の非 UI コンテキストでサーバーの locale に左右されないよう） |
| オプショナルな設定オブジェクト | `@Nullable Options options` | 呼び出し元が省略可能 |
| オプショナルな追加メッセージ | `@Nullable String additionalMessage` | 呼び出し元が省略可能 |
| rootBean | `@Nullable Object rootBean` | rootBean なしで violation を追加した場合（アイテム名の解決が不要な場合）は `null` |
| メッセージ引数 varargs | `@Nullable String... messageArgs` | API 境界として `null` 要素を許容（処理対象の変数の値を出力する用途のため、その値が `null` の場合があることを許容） |

### フィールド

| パターン | 例 | 理由 |
| --- | --- | --- |
| lazy init フィールド | `private @Nullable Field fieldOfBasisPropertyPath` | `initialize()` 呼び出し前は未設定 |
| 条件付き設定フィールド | `private @Nullable String itemNameKeyClassFromAnnotation` | 常にセットされるとは限らない |
| optional 値フィールド | `private @Nullable ValidatorKindEnum validatorKind` | 特定条件のみセット |

### 戻り値

| パターン | 例 | 理由 |
| --- | --- | --- |
| 検索系メソッド（見つからない場合 `null`） | `@Nullable String getFirstFoundEmbeddedVariable(...)` | 変数が存在しない場合は `null` |
| リソース取得（存在しない場合 `null`） | `@Nullable ResourceBundle getResourceBundle(...)` | プロパティファイルが存在しない場合は `null` |

---

## 2. ジェネリクス型引数

### `List` の要素型

`@NullMarked` 下でも Eclipse はジェネリクス型引数を暗黙的に `@NonNull` 補完しないため、
要素型を明示する。

```java
List<@NonNull String>    // 要素が非 null（通常ケース）
List<@Nullable String>   // 要素が null になりうる（例: @Nullable String... から作る場合）
```

### `Map` の型引数

```java
Map<@NonNull String, @Nullable Object>  // EL パラメータ等の値は null になりうる
```

### `Pair` の型引数

null になりうる側の型引数に `@Nullable` を付ける。

```java
@Nullable Pair<@NonNull String, @Nullable String>  // Pair 自体が null になりうる、かつ右側も null になりうる
List<Pair<@Nullable String, String>>               // 左側が null になりうる（文字列リテラルパーツのマーカーとして null を使う設計）
```

---

## 3. ローカル変数への `@NonNull`

JDK 等の非 JSpecify API のメソッド戻り値を受け取るとき、Eclipse は `@NullMarked`
下でも戻り型を "unknown"（`@NonNull` でも `@Nullable` でもない）として扱う。
Eclipse に確定した情報を与えるため、必要に応じてローカル変数に明示的に `@NonNull`
を付ける。

```java
// JDK メソッドの戻り値
@NonNull
String theLeft = propertyPath.substring(...);

// @Nullable フィールドを requireNonNull() 後に non-null として扱う
@NonNull
Field nonNullField = Objects.requireNonNull(nullableField);
```

---

## 4. `ObjectsUtil` パターン

```java
// null 検証して @NonNull に変換
public static <T> @NonNull T requireNonNull(@Nullable T object)

// 要素が nullable な配列・コレクションを受け取るジェネリクス境界
public static <T extends @Nullable Object> T[] requireSizeNonZero(T[] objects)
```

---

## 5. Eclipse 互換性: 意味上は非 null だが `@Nullable` が必要なケース

`jakarta.validation` 等の unannotated インターフェースをオーバーライドする際、
`@NullMarked` コードが継承したパラメータの nullness 制約を強化したと Eclipse が判断し、
"Illegal redefinition of parameter" エラーを報告する。これを抑制するため、実行時には
`null` にならないパラメータにも `@Nullable` を付ける。

対象メソッド:

| インターフェース | メソッド | パラメータ |
| --- | --- | --- |
| `ConstraintValidator` | `initialize()` | アノテーションパラメータ |
| `ConstraintValidator` | `isValid()` | `ConstraintValidatorContext` |
| `ConstraintValidatorFactory` | `getInstance()` | `Class<T>` |
| `ConstraintValidatorFactory` | `releaseInstance()` | `ConstraintValidator<?, ?>` |

理由は各パッケージの `package-info.java` の Javadoc に記載済み。

---

## 6. やらないこと

| やらないこと | 理由 |
| --- | --- |
| フィールド・パラメータ・戻り値に冗長な `@NonNull` を付ける | `@NullMarked` により `@NonNull` が暗黙的に適用されるため |
| Jakarta の `@Nonnull` を使う | JSpecify の `@NonNull` に統一する |
