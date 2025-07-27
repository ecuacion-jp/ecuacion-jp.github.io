# naming convention

## forms, records and entities

These are used mainly in the app with UI like web pages.  

In MVC pattern, Data shown to users is usually put into `form`.
In ecuacion libraries `record` belongs to the `form`, which means `record` is the direct field of the `form` and it stores and conveys data to the view.
`record` has multiple fields used to display values.

`entity` also has multiple fields, but it's used to access database.  

## fields and items

Usually fields is defined as the variable `bean`, `record` or `entity` hold in java world.
On the other hand, `item` is defined as "components" or "controls" in UI apps in ecuacion modules, like text boxes or dropdowns in webapps.

### itemId

itemId specifies 
The form of itemId is like `user.name`, where `user` is usually the name of an entity, and `name` is that of field.
`user` part is called `itemIdClass` and `name` part `itemIdField`.

itemId specifies the item. For example the name of the item is determined by itemId.
Name, ddescription, datatype(String, int,...), maxlength, regular expression, ...


### propertyPath

propertyPath specifies the location of an item. It's a term adopted from Jakarta Validation.  

propertyPath is like this.

- user.dept.manager.name


### itemId and propertyPath

The same item can exist in multiple propertyPaths.  

For example `user` has a relation to `company`, and `dept` also has it. Both `company` is exactly the same.

```
- user.company.name
- user.dept.company.name
```

In this case, both `company` should be the same item, which means that the name of the item is the same.  

Usually these two are not used at the same time, but let's say there's a case.  
In that case, ItemIds of them are `company.name`. `itemId` can be duplicated if they express the same item.

Two item has the same itemId doesn't mean that it's perfectly the same component in the screen.
In the example above, `user.company.name` is a textbox user inputs to update the name, not empty,
on the other hand `user.dept.company.name` is a readonly item.

In this case, you should have two items with the same itemId, but different propertyPath.
items are distinguished by propertyPath, not itemId.


### itemIdForName

Let's consider the following case.

```
- user.name
- user.manager.name
```

`manager` is also a `user`. So these two items have the same item id.
But the displayed label for them should be `user naame` and `manager name`. 
Sometimes you want to change the display name from the original one.
Or you also want to change the name of dropdown items 
because dropdowns usually stores `id` or `code`, but show `name`s for user, so label of it should be the one with `name`, not `id` or `code`.

To resolve the problem, item has a property that expresses itemId for Name.
It can be `itemIdForName`, `itemIdClassForName`, `itemIdFieldForName`.
You can also say `itemIdForDisplayName`, but `itemIdForName` is recommended because the latter is long.


### itemId sample

Let's use these classes as an example.
```
public class SomeForm {
  private ParentRecord parentRecord = new ParentRecord();

  // getter, setter
}

private ParentRecord {
  private String name;
  private ChildRecord childRecord;
  private ChildRecord siblingsChildRecord;
  
  // getter, setter
}

private ChildRecord {
  private String name;
  
  // getter, setter
}
```

itemId : `parentRecord.name`, `childRecord.name`, `siblingsChildRecord.name`

Even if the same class `ChildRecord` is used, itemId differs between `childRecord.name` and `siblingsChildRecord.name`
because they are displayed as different components.

### itemName

A localized name for items. It's usually displayed as a component name on a screen.


## others

- fieldId : `name`, `childRecord.name`, `siblingsChildRecord.name`
  (It usually specifies the field of the root record. 
   But sometimes it also specifies the field of `childRecord` 
   when there's no form and only `childrecord` exists in the context.)

- recordId : `parentRecord`, `parentRecord.childRecord`, `parentRecord.siblingsChildRecord`  
  (The recordName sounds more like `childRecord`, but `childRecord` is not used in codes.)

- rootRecordId : `parentRecord`
  (root means direct property of the form)


### Name

- In the library localized names are called `displayName`.

- `recordDisplayName`, `fieldDisplayName`, `itemDisplayName`, `displayName` sounds okay, but to unify in this library `recordDisplayName` and  `fieldDisplayName` are not used. `itemDisplayName` is the official name, but since it's a bit long you can use `itemName` or `displayName`.
  (`itemName` can be misunderstood as an html name of the item in some context, so consider the context.)
