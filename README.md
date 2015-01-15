Android Essentials Toolbox
==========================

The Android Essentials Toolbox is an Atlas toolbox containing logic for Android Permissions, Manifest, and other Android domain specific logic useful in program analysis.

For more details see [https://ensoftcorp.github.io/android-essentials-toolbox](https://ensoftcorp.github.io/android-essentials-toolbox).

## Overview
This feature contains encoded mappings recovered from the Android source code (as of API version 19) to map [Protection Levels](https://developer.android.com/guide/topics/manifest/permission-element.html#plevel) and [Permission Groups](https://developer.android.com/guide/topics/manifest/permission-element.html#pgroup) to the corresponding [manifest permissions](https://developer.android.com/guide/topics/manifest/permission-element.html).  Permission details (simple name, quaified name, description, etc.) are encoded from information found in the [Google documentation](https://developer.android.com/reference/android/Manifest.permission.html). Using the [University of Toronto's PScout utility](http://pscout.csl.toronto.edu/) this feature also includes mappings from permissions to Android permission restricted API methods.

## Setup
The `com.ensoftcorp.open.android.essentials` project is an Eclipse plugin that can be installed into the Eclipse environment.  To install the Eclipse plugin from the workspace right click on the project and navigate to `Export`->`Plug-in Development`->`Deployable plug-ins and fragments`.  Select `Next` and make sure only the `com.ensoftcorp.open.android.essentials` project is selected.  Then select the `Install into host.` radio and click `Finish`.  You will need to restart Eclipse.

Alternatively if you have a toolbox project that depends on the Android Essentials Toolbox keeping both projects in the workspace is enough, but you should note that confusion may occur if there is an installed version and a version in the workspace.  The Atlas interpreter tends to give priority to the version in the workspace.

## Example Usage

    int apiVersion = 19;
    PermissionMapping.applyTags(apiVersion);
    Q internetRestrictedAPIs = PermissionMapping.getMethods(Permission.INTERNET, apiVersion);
    long numMethods = CommonQueries.nodeSize(internetRestrictedAPIs);
    System.out.println("There are " + numMethods + " restricted methods for " + Permission.INTERNET.getQualifiedName());

## Maintenance
This section provides some tips on performing maintenance on this project.

### Updating Permissions and Permission Mappings
The Permission, PermissionGroup, and ProtectionLevel objects are primarily generated by scraping together information available in the online Android documentation and the AndroidManifest.xml file found in the Android source.  Using both sources allows us to gather both documented and undocumented permissions information.  The partial code generators are located in the `AndroidPermissionGenerator` Eclipse project, which has a project dependency on the `com.ensoftcorp.open.android.essentials` project.  The primary resources are listed below.

- https://developer.android.com/reference/android/Manifest.permission.html
- https://developer.android.com/reference/android/Manifest.permission_group.html
- https://developer.android.com/guide/topics/manifest/permission-element.html
- https://developer.android.com/reference/android/content/pm/PermissionInfo.html
- https://raw.githubusercontent.com/android/platform_frameworks_base/master/core/res/AndroidManifest.xml

#### Updating Permissions
Permissions come in two flavors (documented and undocumented).  A documented permission exists in the constants table at [https://developer.android.com/reference/android/Manifest.permission.html](https://developer.android.com/reference/android/Manifest.permission.html).  An undocumented permission is not present in the constants table, but is referred to in the AndroidManifest.xml file at [https://raw.githubusercontent.com/android/platform_frameworks_base/master/core/res/AndroidManifest.xml](https://raw.githubusercontent.com/android/platform_frameworks_base/master/core/res/AndroidManifest.xml).

1) To generate the documented permissions, run the main method of the `GenerateDocumentedPermissions` class.  If successful the class prints to stdout a series of Permission instantiations that can be cut and pasted into the `Permission` class.  The `GenerateDocumentedPermissions` also prints code to add each Permission instantiation to the `allDocumentedPermissions` collection, which needs to be cut and pasted into the appropriate place in the `Permission` class as well.

2) To generate the undocumented permissions, run the main method of the `GenerateUndocumentedPermissions` class.  The output and process for updating the code in the `Permission` class is the same as updating the documented permissions.
IMPORTANT: Generate and update the documented permissions first!  The code for generating undocumented permissions was lazily written and just uses the updated documented permissions list to identify undocumented permissions in the AndroidManifest.xml Android source file.  This code could be rewritten to break this dependency...someday.

#### Updating Permission Groups and Mappings
Like permissions, permission groups come in two flavors (documented and undocumented).  A documented permission group exists in the constants table at [https://developer.android.com/reference/android/Manifest.permission_group.html](https://developer.android.com/reference/android/Manifest.permission_group.html).  An undocumented permission group is a permission group that does not exist in the constants table, but is referred to in the AndroidManifest.xml file at [https://raw.githubusercontent.com/android/platform_frameworks_base/master/core/res/AndroidManifest.xml](https://raw.githubusercontent.com/android/platform_frameworks_base/master/core/res/AndroidManifest.xml).  However at the time of this writing, no undocumented permission groups were discovered to exist.

To generate the documented permission groups run the main method of the `GenerateDocumentedPermissionGroups` class.  The class writes to stdout and outputs a series of PermissionGroup instantiations and a collection just like the `GenerateDocumentedPermissions` and `GenerateUndocumentedPermissions` classes.  You should copy and paste the results, updating the appropriate places in the `PermissionGroup` class.

To generate the mapping between a permission group and permissions run the main method of the `GeneratePermissionGroupToPermissionMapping` class.  Copy the two outputs written to stdout and paste them in the appropriate places in the `PermissionGroup` class.

#### Updating Protection Levels and Mappings
Since there are only a few protection levels and the Android documentation only covers the 4 main protection levels (normal, dangerous, signature, and signatureOrSystem), the information for protection levels is gathered solely from the AndroidManifest.xml Android source file.

To generate the mapping between a protection level and permissions run the main method of the `GenerateProtectionLevelToPermissionMapping` class.  Copy the two outputs written to stdout and paste them in the appropriate places in the `ProtectionLevel` class.  If a new protection level has been introduced (not likely) you will be responsible for adding it manually to fix the compiler errors for the missing protection level type.

#### Running Sanity Checks (Unit Tests)
After updating the Permission, PermissionGroup, and ProtectionLevel classes it is highly recommended to the `SanityChecks` JUnit tests and fix any errors.

#### Updating Protected API Method Permission Mappings
TODO
