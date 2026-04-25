# checked/unchecked 例外 → Violations 移行ガイド

`jp.ecuacion.lib.core.exception.checked` および `unchecked` パッケージの例外クラスが
`@Deprecated` になったため、`BusinessViolation` / `Violations` に置き換える。

## ライブラリ情報

| 旧 | 新 |
|---|---|
| `jp.ecuacion.lib.core.exception.checked.BizLogicAppException` | `jp.ecuacion.lib.core.violation.BusinessViolation` |
| `jp.ecuacion.lib.core.exception.checked.MultipleAppException` | `jp.ecuacion.lib.core.violation.Violations` |
| `jp.ecuacion.lib.core.exception.checked.SingleAppException` | （不要） |
| `jp.ecuacion.lib.core.exception.checked.AppException` | （不要。catch は `ViolationException` へ） |
| `jp.ecuacion.lib.core.exception.checked.AppWarningException` | `jp.ecuacion.lib.core.exception.ViolationWarningException` |
| `jp.ecuacion.lib.core.exception.checked.ValidationAppException` | `jp.ecuacion.lib.core.violation.Violations` |
| `jp.ecuacion.lib.core.exception.unchecked.AppRuntimeException` | （不要。`ViolationException` は `RuntimeException`） |
| `jp.ecuacion.lib.core.exception.unchecked.EclibRuntimeException` | `RuntimeException` |

`ViolationException`（`Violations.throwIfAny()` がスローする例外）は `RuntimeException` のサブクラスなので、
deprecated 例外の `throws` 宣言は全て削除できる。

---

## パターン1: 基本の throw

```java
// 旧
throw new BizLogicAppException("MSG_ID");
throw new BizLogicAppException("MSG_ID", "arg1", "arg2");

// 新
new Violations().add(new BusinessViolation("MSG_ID")).throwIfAny();
new Violations().add(new BusinessViolation("MSG_ID", "arg1", "arg2")).throwIfAny();
```

## パターン2: itemPropertyPaths 付き throw

```java
// 旧
throw new BizLogicAppException(new String[] {"fieldName"}, "MSG_ID");
throw new BizLogicAppException(new String[] {"fieldName"}, "MSG_ID", "arg1");

// 新
new Violations().add(new BusinessViolation(new String[] {"fieldName"}, "MSG_ID")).throwIfAny();
new Violations().add(
    new BusinessViolation(new String[] {"fieldName"}, "MSG_ID", "arg1")).throwIfAny();
```

## パターン3: throws 宣言の削除

```java
// 旧
public void foo() throws BizLogicAppException { ... }
public void foo() throws BizLogicAppException, Exception { ... }

// 新
public void foo() { ... }
public void foo() throws Exception { ... }
```

## パターン4: 複数エラーをまとめてスロー（MultipleAppException）

```java
// 旧
List<SingleAppException> exList = new ArrayList<>();
if (conditionA) {
  exList.add(new BizLogicAppException(new String[] {"fieldA"}, "MSG_A", argA));
}
if (conditionB) {
  exList.add(new BizLogicAppException(new String[] {"fieldB"}, "MSG_B"));
}
if (exList.size() > 0) {
  throw new MultipleAppException(exList);
}

// 新
Violations violations = new Violations();
if (conditionA) {
  violations.add(new BusinessViolation(new String[] {"fieldA"}, "MSG_A", argA));
}
if (conditionB) {
  violations.add(new BusinessViolation(new String[] {"fieldB"}, "MSG_B"));
}
violations.throwIfAny();
```

## パターン5: catch ブロックで cause を参照していた場合

旧来、`AppRuntimeException` の cause として `BizLogicAppException` をラップしていたコードがある場合。
新実装では `ViolationException` が直接スローされるため、catch 対象を変更する。

```java
// 旧
try {
  someMethod(); // 内部で AppRuntimeException(new BizLogicAppException(...)) をスロー
} catch (AppRuntimeException ex) {
  if (ex.getCause() instanceof BizLogicAppException) {
    BizLogicAppException blae = (BizLogicAppException) ex.getCause();
    throw new BizLogicAppException(new String[] {"fieldA", "fieldB"}, blae.getMessageId());
  } else {
    throw ex;
  }
}

// 新
try {
  someMethod(); // 新実装では ViolationException を直接スロー
} catch (ViolationException ex) {
  // itemPropertyPath を付け直して re-throw
  Violations newViolations = new Violations();
  for (BusinessViolation bv : ex.getViolations().getBusinessViolations()) {
    newViolations.add(new BusinessViolation(
        new String[] {"fieldA", "fieldB"}, bv.getMessageId()));
  }
  newViolations.throwIfAny();
}
```

## パターン6: AppWarningException の throw

ユーザーへの確認ポップアップを表示する警告例外。

```java
// 旧
throw new AppWarningException("MSG_ID");
throw new AppWarningException("MSG_ID", "arg1");

// 新
new Violations().add(new BusinessViolation("MSG_ID")).throwWarningIfAny();
new Violations().add(new BusinessViolation("MSG_ID", "arg1")).throwWarningIfAny();
```

## パターン7: ValidationAppException の throw

`ConstraintViolation` を1件の例外として扱っていた場合。

```java
// 旧
throw new ValidationAppException(constraintViolation);

// 新
new Violations().add(constraintViolation).throwIfAny();
```

## パターン8: AppRuntimeException の throw

`AppRuntimeException` は `AppException`（checked）を `RuntimeException` としてスローするための
ラッパーだったが、`ViolationException` は既に `RuntimeException` のため不要になった。

```java
// 旧
throw new AppRuntimeException(new BizLogicAppException("MSG_ID"));

// 新（AppRuntimeException は不要）
new Violations().add(new BusinessViolation("MSG_ID")).throwIfAny();
```

## パターン9: EclibRuntimeException の throw

```java
// 旧
throw new EclibRuntimeException("message");
throw new EclibRuntimeException(cause);
throw new EclibRuntimeException("message", cause);

// 新
throw new RuntimeException("message");
throw new RuntimeException(cause);
throw new RuntimeException("message", cause);
```

## パターン10: AppException / AppRuntimeException の catch

```java
// 旧
} catch (AppException ex) { ... }
} catch (AppRuntimeException ex) { ... }

// 新
} catch (ViolationException ex) { ... }
```

## パターン11: 戻り値のあるメソッドで throwIfAny() を使う場合

`throwIfAny()` は常に例外をスローするが、コンパイラはその事実を認識できない。
戻り値のあるメソッドで `throwIfAny()` の後に `return` が必要になる場合は、
`return null` ではなく `throw new ViolationException(violations)` を置く。
こうすることで「ここは到達不能だが、意図的にエラーを投げる」という意図が明確になる。

```java
// 旧
} else {
  throw new RuntimeException(new BizLogicAppException("MSG_ID", arg));
}

// 新
} else {
  Violations violations = new Violations().add(new BusinessViolation("MSG_ID", arg));
  violations.throwIfAny();
  throw new ViolationException(violations); // コンパイラ向け（到達不能）
}
```

---

## パターン12: ValidationUtil の置き換え

`ValidationUtil` クラス自体が `@Deprecated` になったため、`Violations` を直接使用する。

```java
// 旧
ValidationUtil.validateThenThrow(object,
    ValidationUtil.messageParameters()
        .isMessageWithItemName(true)
        .messagePostfix(Arg.message(msgId, args)));

// 新
new Violations()
    .addAll(Validation.buildDefaultValidatorFactory().getValidator().validate(object))
    .messageParameters(Violations.newMessageParameters()
        .isMessageWithItemName(true)
        .messagePostfix(Arg.message(msgId, args)))
    .throwIfAny();
```

## パターン13: WebAppWarningException / throwWarning() の置き換え

`WebAppWarningException` が deprecated になり、`SplibGeneralService#throwWarning()` の引数も変わった。
`Violations#throwWarningIfAny()` を直接使用して、確認済みメッセージのチェックも明示的に記述する。

```java
// 旧
public void warning(SomeForm form) throws WebAppWarningException {
  throwWarning(form.getConfirmedWarningMessageSet(), "groupId", null, "MSG_ID_1");
  throwWarning(form.getConfirmedWarningMessageSet(), "groupId", null, "MSG_ID_2");
}

// 新（ViolationWarningException は RuntimeException のため throws 宣言不要）
public void warning(SomeForm form) {
  if (!form.getConfirmedWarningMessageSet().contains("MSG_ID_1")) {
    new Violations().add("MSG_ID_1").throwWarningIfAny();
  }
  if (!form.getConfirmedWarningMessageSet().contains("MSG_ID_2")) {
    new Violations().add("MSG_ID_2").throwWarningIfAny();
  }
}
```

---

## import チートシート

```java
// 削除するもの
import jp.ecuacion.lib.core.exception.checked.AppException;
import jp.ecuacion.lib.core.exception.checked.AppWarningException;
import jp.ecuacion.lib.core.exception.checked.BizLogicAppException;
import jp.ecuacion.lib.core.exception.checked.MultipleAppException;
import jp.ecuacion.lib.core.exception.checked.SingleAppException;
import jp.ecuacion.lib.core.exception.checked.ValidationAppException;
import jp.ecuacion.lib.core.exception.unchecked.AppRuntimeException;
import jp.ecuacion.lib.core.exception.unchecked.EclibRuntimeException;
import jp.ecuacion.lib.core.util.ValidationUtil;
import jp.ecuacion.splib.web.exception.WebAppWarningException;

// 追加するもの（使用する場合のみ）
import jakarta.validation.Validation;                             // ValidationUtil 置き換え時のみ
import jp.ecuacion.lib.core.exception.ViolationException;         // catch する場合のみ
import jp.ecuacion.lib.core.exception.ViolationWarningException;  // throwWarningIfAny() を catch する場合のみ
import jp.ecuacion.lib.core.violation.BusinessViolation;
import jp.ecuacion.lib.core.violation.Violations;
```

import はアルファベット順を維持すること（Google Java Style Guide）。

---

## 作業手順

1. 対象ファイルを抽出する

   ```bash
   grep -rln \
     "AppException\|AppWarningException\|BizLogicAppException\|MultipleAppException\
   \|SingleAppException\|ValidationAppException\|AppRuntimeException\|EclibRuntimeException\
   \|ValidationUtil\|WebAppWarningException" \
     src/ --include="*.java"
   ```

2. 各ファイルについて上記パターンに従って変換する

3. ビルド検証を実行する

   ```bash
   mvn checkstyle:check spotbugs:check
   mvn javadoc:javadoc
   ```

## 注意事項

- コメントアウトされた `// throw new BizLogicAppException(...)` 等は変換しない
- メソッドの throws 宣言から deprecated 例外を削除した結果、
  他の checked exception も宣言がなくなる場合はコンパイルエラーになるので注意

---

# `${+...}` → `#{...}` プロパティ参照構文の変更

`*.properties` ファイル内で他のプロパティファイルのキーを参照する構文が変わった。

## 変換ルール

| 旧 | 新 |
|---|---|
| `${+messages:xxx}` | `#{messages:xxx}` |
| `${+item_names:xxx}` | `#{item_names:xxx}` |
| `${+strings:xxx}` | `#{strings:xxx}` |
| `${+enum_names:xxx}` | `#{enum_names:xxx}` |
| `${+application:xxx}` | `#{application:xxx}` |

また、ファイル種別を省略した `#{xxx}` という新構文も使用可能になった。
その場合は messages, item_names, strings, enum_names の順に検索する。

## 対象ファイルの抽出

```bash
grep -rln '\${+' src/ --include="*.properties"
```

## 作業手順

1. 上記コマンドで対象ファイルを抽出する
2. `${+` を `#{` に、`${+` 直後の `+` を削除する形で置換する
   - `${+messages:` → `#{messages:`
   - `${+item_names:` → `#{item_names:`
   - （他の fileKind も同様）
3. ビルド検証を実行する

   ```bash
   mvn checkstyle:check spotbugs:check
   mvn javadoc:javadoc
   ```
