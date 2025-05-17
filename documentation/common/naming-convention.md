# naming convention

## forms, records and entities

These are used mainly in the app with UI like web pages.  

In MVC pattern, Data shown to users is usually put into `form`.
In ecuacion libraries `record` belongs to the `form`, which means `record` is the direct field of the `form` and it stores and conveys data to the view.
`record` has multiple fields used to display values.

`entity` also has multiple fields, but it's used to access database.  

## fields and items

Usually fields is defined as the variable `record` or `entity` hold in java world.
On the other hand, `item` is defined in ecuacion modules as "components" or "controls" in UI apps, like text boxes or dropdowns in webapps.

### itemId

The form of itemId is like `user.name`, where `user` is usually the name of an entity, and `name` is that of field.
`user` part is called `itemIdClass` and `name` part is called `itemIdField`.

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

A localized name for items. It's usually displayed as a component name.


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
