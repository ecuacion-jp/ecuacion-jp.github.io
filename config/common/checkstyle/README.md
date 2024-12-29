# checkstyle config

## google_checks.xml

- Obtained on 2024/12/29, at commit `d5efae6` from https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml  
(Updates while unknown are not very preferable, so the copied file is referred.)

## Suppression Filters

There are some kinds of suppression filters for `checkstyle`.

1. suppression-filter-for-libs-and-utils.xml
    - Used for public (means the repository is in the state of public in `github`) repositories like `ecuacion-lib`, `ecuacion-util`, `ecuacion-splib` and so on
    - The strictest checks are adapted

1. suppression-filter-for-private-apps.xml
    - Used for apps.
    - Checks on `javadoc` are suppressed.

## How To Use with Eclipse

- When you use checkstyle on eclipse, right-click the project -> Maven -> Update Project will set the checkstyle settings on eclipse (.checkstyle).

- ALERT: UPDATE `eclipse-cs` plugin to the latest version. Version 10.21.1 or higher needed.
