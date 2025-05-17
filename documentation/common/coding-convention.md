# Coding Conventions

## Treatment of `null`

### Background

- Some say arguments in constructors and methods should be considered not to accept `null`s.
  - Allowing `null` complicates the understanding of the method.
  - Add `@Nonnull` or `@Nullable` to arguments always.
- Some say return values in methods should not `null`.
  - By returning `null`s Callers of the mothods have to check if the return value is `null`. That complicates your code.
  - In most cases you'd better throw Exceptions.
  - Add `@Nonnull` or `@Nullable` to return value always.
  - Null Object canllWe can also see "Null-Object pattern" which returns not `null`, but the object which expresses returning nothing as the solution, but I don't understand why it's better than returning `null` (Optional.EMPTY or Employee.ANONYMOUS).  
Because we have to check if the return value is `null object`...)

### What Ecuacion Thinks

#### About Returning `null`

- Basically it's not good. By returning `null` without throwing `Exception` may cause incorrect resulting value.
- Returning `null` seems allowable only in the following cases, 
  1. Getter methods in data containers (like entities) is allowed to be `@Nullable`.
     -> It doesn't mean it's recommended. It's not the best choice, 
        but it's so troublesome to use `Optional` return values for the getters of nullable fields.

- In other cases returning `null` doesn't seem to make sense.
  - Consider `Optional` instead of `null` in the following cases.
    1. Getter method cannot get the result, may be no data found.
     -> In most cases you need to check whether the return value is `null`. 
        Then it's better to return `Optional` because by using it you cannot forget to check.
        (Actually if you do the null check with Optional#isPresent() and then use get(), it's the same as using `null`, 
        or even worse because it uses more memory resource. 
        Since `get()` is unsafe (throws NoSuchElementException), it doesn't make any change as long as it's used.
        `NPE` (or `NoSuchElementException`) is bad because it causes Exception at runtime and it takes time to find the right way to fix.
        So basically NEVER USE `get()` and use safe methods like `ifPresent`, `map`, `orElse`, etc...
        As a rule, You can use `get()` only when you know it cannot be `null`. (It's better than null because you can forget null check but you cannot forget treatment of Optional.))
        
  - Set `@Nonnull` when the datatype of the return value is `Collection`.
    -> Returned `Collection` usually processed with a loop. If you return `null` the caller have to check whether it's `null` but if you return `Collection` with size zero the caller doesn't have to check whether it's `null` and the caller is able to loop it without thinking `null` risk.

- The case that the function of a method is to change the format of the value specified with argument, basically don't allow `null` input or output.
  -> It's not allowed for now but I'm thinking maybe there might be a case in the future...
  -> This method doesn't increase number of `null`s. It's just returning incoming values. In that meaning this is not very bad.
  -> You can say it's good for coding because it abbreviates `value == null ? null : ` part from the following code.
     But at the same time `null` is not treated specially with this implementation. `null` should always be noticed and considered as a special value I think...
     
           ```java
             String str = value == null ? null : someMethod(value);
           ```  

#### About `null` arguments

- Basically it happens.
- Methods with many arguments tend to accept multiple `null` arguments. 
  That does not seem very beautiful, it's better to consider builder pattern or something, 
  but no rules are established by ecuacion with that.

#### About using `@Nonnull` or `@Nullable` to Return Values

- It needs to use.
- The rule can be like "No annotation is equal to @Nullable", 
  but without annotation we cannot be sure it's considered once or not.
  So we put `@Nullable` to the parameter to prove it's considered.
- Primitive datatypes are always nonnull. You don't need to put annotations to them.

#### About using `@Nonnull`, `@RequireNonnull` or `@Nullable` to Arguments

- When you use `@Nonnull`, IDE checks if the argument is reaaly nullable or not, and shows warnings if not.  
  The problem is, it shows warning when it's not sure the value is nonnull, 
  which means warnings are shown even if the value is actually always nonnull.
- To remove the warnings, you need to use  `Objects.requireNonnull(...)` or `ObjectsUtil.paramRequireNonnull(...)` (recommended) right before the method calls. That's **annoying** actually.
- To show that the argument need to be nonnull without letting IDEs show warnings, `@RequireNonnull (jp.ecuacion.lib.core.annotation)` annotation is introduced in `ecuacion-lib`.  
- With using it you don't have to add any codes, and the goal achieved because it tells other developers that it was considered which annotation should be added. 
- Since `@Nonnull` is annoying like I said before, the situation Using `@Nonnull` should be limited. 
  The situation `@Nonnull` needs to be used is as follows:
  - "Arbitrary Number of Arguments" like `String... args`. They are always nonnull as Java language.
- In other cases for now we don't see the case we need to use `@Nonnull` rather than `@RequireNonnull`. 
  Until the case is found, let's use `@RequireNonnull` always.
- `@Nullable` is also needed when the argument is nullable.

### Ecuacion Rules

- Always put one of `@Nonnull` (only to "Arbitrary Number of Arguments"), `@RequireNonnull` or `@Nullable` to all the arguments, 
  and `@Nonnull` or `@Nullable` to return values of methods and constructors.
- When you use `@RequireNonnull` to arguments, make sure it's really nonnull by using `ObjectsUtil.paramRequireNonnull(...)` inside the methods or the constructors.
- You can use `@Nullable` to return values only when the usage is compliant to the following.
  - Getter methods in data containers (like entities) is allowed to be `@Nullable`
