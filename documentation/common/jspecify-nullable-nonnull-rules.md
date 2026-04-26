# JSpecify @Nullable / @NonNull Rules

## Prerequisites

- All packages are annotated with `@NullMarked` in `package-info.java`.
- Under `@NullMarked`, unannotated types are treated as `@NonNull` by default.
- Exception: packages containing Java annotation definitions use `@NullUnmarked`
  (null analysis is unnecessary for annotation types).

---

## 1. Where to Add `@Nullable`

### Parameters

| Pattern | Example | Reason |
| --- | --- | --- |
| `Locale` parameter | `@Nullable Locale locale` | Substituted with `Locale.ROOT` when `null` (avoids server-locale dependency in batch/non-UI contexts) |
| Optional configuration object | `@Nullable Options options` | May be omitted by the caller |
| Optional additional message | `@Nullable String additionalMessage` | May be omitted by the caller |
| Root bean | `@Nullable Object rootBean` | `null` when no root bean is provided (i.e. item name resolution is not required) |
| Message argument varargs | `@Nullable String... messageArgs` | Allows `null` elements at the API boundary — callers may pass variable values being displayed, which can themselves be `null` |

### Fields

| Pattern | Example | Reason |
| --- | --- | --- |
| Lazy-initialized field | `private @Nullable Field fieldOfBasisPropertyPath` | Not yet set before `initialize()` is called |
| Conditionally set field | `private @Nullable String itemNameKeyClassFromAnnotation` | Not always assigned |
| Optional-value field | `private @Nullable ValidatorKindEnum validatorKind` | Set only under specific conditions |

### Return Values

| Pattern | Example | Reason |
| --- | --- | --- |
| Search method (returns `null` when not found) | `@Nullable String getFirstFoundEmbeddedVariable(...)` | Returns `null` when the variable does not exist |
| Resource acquisition (returns `null` when absent) | `@Nullable ResourceBundle getResourceBundle(...)` | Returns `null` when the properties file does not exist |

---

## 2. Generic Type Arguments

### `List` Element Type

Eclipse does not implicitly apply `@NonNull` to generic type arguments even under
`@NullMarked`, so the element type must be stated explicitly.

```java
List<@NonNull String>    // elements are non-null (the common case)
List<@Nullable String>   // elements may be null (e.g. a List built from @Nullable String...)
```

### `Map` Type Arguments

```java
Map<@NonNull String, @Nullable Object>  // values such as EL parameters may be null
```

### `Pair` Type Arguments

Annotate whichever type argument may be `null`.

```java
@Nullable Pair<@NonNull String, @Nullable String>  // Pair itself may be null; right side may be null
List<Pair<@Nullable String, String>>               // left side may be null (used as a literal-part marker)
```

---

## 3. `@NonNull` on Local Variables

When the right-hand side is a JDK or other non-JSpecify API method, Eclipse treats
the return type as "unknown" (neither `@NonNull` nor `@Nullable`) even under
`@NullMarked`. Annotate the local variable explicitly to give Eclipse definitive
information.

```java
// Return value of a JDK method
@NonNull
String theLeft = propertyPath.substring(...);

// Narrowing a @Nullable field to non-null after requireNonNull()
@NonNull
Field nonNullField = Objects.requireNonNull(nullableField);
```

---

## 4. `ObjectsUtil` Patterns

```java
// Validates non-null and converts to @NonNull
public static <T> @NonNull T requireNonNull(@Nullable T object)

// Generic bound for arrays/collections whose elements may be nullable
public static <T extends @Nullable Object> T[] requireSizeNonZero(T[] objects)
```

---

## 5. Eclipse Compatibility: `@Nullable` on Semantically Non-Null Parameters

When overriding methods from unannotated interfaces (such as `jakarta.validation`),
Eclipse reports an "Illegal redefinition of parameter" error if `@NullMarked` code
strengthens the nullness constraint of an inherited parameter. To suppress this,
`@Nullable` is added even though the parameter is never actually `null` at runtime.

Affected methods:

| Interface | Method | Parameter |
| --- | --- | --- |
| `ConstraintValidator` | `initialize()` | annotation parameter |
| `ConstraintValidator` | `isValid()` | `ConstraintValidatorContext` |
| `ConstraintValidatorFactory` | `getInstance()` | `Class<T>` |
| `ConstraintValidatorFactory` | `releaseInstance()` | `ConstraintValidator<?, ?>` |

The rationale is documented in the Javadoc of each affected `package-info.java`.

---

## 6. What NOT to Do

| Do not | Reason |
| --- | --- |
| Add redundant `@NonNull` to fields, parameters, or return values | `@NullMarked` already implies `@NonNull` |
| Use Jakarta's `@Nonnull` | Use JSpecify's `@NonNull` exclusively |
