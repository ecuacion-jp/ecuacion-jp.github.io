# naming convention

## IDs of records & fields

These are used mainly the app with UI like web pages.  

In MVC pattern, Data shown to users is usually put into `form`.
In ecuacion libraries `record` belongs to the `form`, which means `record` is the direct field of the `form` and it stores and conveys data to the view.

Web apps for example, each page has "components" or "controls" like text boxes and pulldowns. 
This part describes the name convention of those components in web pages, or records and field in them which belong to a `form`.

Use these classes as an example.
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

- fieldId (/ rootRecordFieldId) : `name`, `childRecord.name`, `siblingsChildRecord.name`
  (It usually specifies the field of the root record. 
   But sometimes it also specifies the field of `childRecord` 
   when there's no form and only `childrecord` exists in the context.)

- recordId : `parentRecord`, `parentRecord.childRecord`, `parentRecord.siblingsChildRecord`  
  (The recordName sounds more like `childRecord`, but `childRecord` is not used in codes.)

- rootRecordId : `parentRecord`
  (root means direct property of the form)

- itemId (/ propertyPath): `parentRecord.name`, `parentRecord.childRecord.name`, `parentRecord.siblingsChildRecord.name`  
  (The path from the property name defined in form to the property name. It specifies the property location.
   `propertyPath` is taken from `ConstraintViolation`. 
   It's used to specify the error items on the screen.)  

- [item]displayNameId : `parentRecord.name`, `childRecord.name`  
  (the name which designates the record class and its property. 
   It's used to obtain display name from `field_name.properties`.
   `siblingsChildRecord.name` is not the itemName because `siblingsChildRecord` is not a record class name.)

