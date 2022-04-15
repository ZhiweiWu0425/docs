---
subtitle: Find out the purpose of each directory in the October CMS root folder
---
# Directory Structure

October CMS uses a modular structure where most programming features are found in either the modules or plugins directory.

## Bootstrap Directory

The `bootstrap` directory contains the app.php bootstrap script that loads the Laravel framework. The directory also contains the custom PHP class autoloader. Typically, files in this directory should not be modified.

## Config Directory

The `config` directory contains all the application configuration files. Each file controls how the application functions. Configuration files can be changed according to your application requirements. System updates do not modify the configuration files.

## Plugins Directory

The `plugins` directory packages that extend the core functionality of October CMS. Plugins can modify the platform by introducing new features. By default, the system loads all plugins found in the filesystem. Specific plugins can be disabled using the `system.disable_plugins` configuration parameter.

## Storage Directory

The `storage` directory contains log files, cache files, sessions and other files generated by October CMS. It includes several subdirectories:

* `app` - contains application-specific storage files, such as media files, file uploads and automatically generated resources, e.g. resized files and combined asset files.
* `framework` - used by the Laravel framework to store its generated files and caches.
* `cms` - used by the October CMS platform to store its generated files and caches.
* `logs` - contains the application's log files.
* `temp` - used to store temporary application files.

## Modules Directory

The `modules` directory contains the core packages of October CMS, providing core functionality that is common across the system. By default, modules are loaded automatically based on their presence in the file system. However, it is possible to specify which modules to load with the `system.load_modules` configuration parameter. The `system` module must be loaded at a minimum for the application to operate.

* `system` - defines the core functionality of October CMS and is a required module.
* `cms` - introduces functionality for rendering the frontend website theme. It is responsible for routing requests to pages, rendering
* `pages` and handling AJAX requests.
* `backend` - responsible for displaying the Administration Area pages.
* `editor` - introduces a user interface for editing CMS templates in the Administration Area.
* `media` - introduces a media file management user interface in the Administration Area. It allows back-end users to upload media files like images or video files and include them in other places, for example, blog posts.
* `tailor` - introduces the October CMS Tailor features.

## Themes Directory

The `themes` directory contains subdirectories for front-end website CMS themes. CMS themes include template files for the website pages, layouts, partials, assets, and other files. The active theme is set using the `cms.active_theme` configuration parameter and can be overridden from the Administration Area Settings page.

## Vendor Directory

The `vendor` directory contains packages that are included via Composer. Some plugins can include the vendor directory as well. The system vendor directory takes priority over plugin-specific vendor directories.