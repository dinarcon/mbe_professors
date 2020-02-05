# Drupal 8 Migrations by Example

A demo module created by [Mauricio Dinarte](https://www.drupal.org/u/dinarcon) ([@dinarcon](https://twitter.com/dinarcon)) to explain migrations concepts in Drupal 8.

This module has been used in conference talks to explain migration concetps. Find more information at:

* http://bit.ly/migrations-by-example - Slide deck.
* https://www.youtube.com/watch?v=eBP2vQIwx-o - Recording. It is old and does not uses the latest code in the repo, but the concepts still apply.
* https://gist.github.com/dinarcon/72922e8cf634ada47483117ffa659d76 - JSON source migration example.

## Dependencies

The following projects are required to run this demo. The version number indicates which version was last used for testing.

* [Drupal](https://www.drupal.org/project/drupal) 8.8.2
* [Address](https://www.drupal.org/project/address) 8.x-1.7
* [Entity reference revisions](https://www.drupal.org/project/entity_reference_revisions) 8.x-1.7
* [Migrate plus](https://www.drupal.org/project/migrate_plus) 8.x-5.0-rc4
* [Migrate source csv](https://www.drupal.org/project/migrate_source_csv) 8.x-3.2
* [Migrate tools](https://www.drupal.org/project/migrate_tools) 8.x-4.5
* [Paragraphs](https://www.drupal.org/project/paragraphs) 8.x-1.10
* [Drush](https://github.com/drush-ops/drush) 10.2.0

## Examples

This demo includes 3 migration configurations.

* mbe_book_paragraph for importing data into paragraphs entities. There are no dependencies on other migrations.
* mbe_photo_field for importing data into file entities. There are no dependencies on other migrations.
* mbe_professors for importing data into node entities. This depend on other migrations to be run before: mbe_book_paragraph and mbe_photo_field.

## Instructions

* Install module dependencies via composer: `composer require 'drupal/paragraphs:^1.10' 'drupal/address:^1.7' 'drupal/migrate_plus:^5.0' 'drupal/migrate_source_csv:^3.2' 'drupal/migrate_tools:^4.5'`
* If you do not have Drush available, install the latest version via composer: `composer require drush/drush`. After this step, you may call it via `./vendor/bin/drush`.
* Make sure that your Drupal installation has a `/modules/custom` folder. The `modules` folder should exist, but the `custom` sub-folder might not. Create it if needed.
* Download the demo module contained in this repository into the `/modules/custom` folder. You can do this by cloning this repository or [downloading a ZIP file](https://github.com/dinarcon/mbe_professors/archive/master.zip). **Important:** The name of the folder containing this demo must ve `mbe_professors`. If you get it from the ZIP file the folder might be named `mbe_professors-master`. If that is the case, rename the folder to `mbe_professors` to prevent errors reading the CSV files.
* Verify that the CSV files are in the proper location. See instructions below.
* Enable the Professors Example Migration (`mbe_professors`) module.
* Run migrations using Drush. See instructions below.

## CSV files location

The `path` configuration for the CSV source plugin accepts an absolute path or relative path from the Drupal installation folder.

The examples use a relative path and it is assumed that you place this demo module in a `modules/custom` folder. Therefore, the CSV files will be located at `modules/custom/mbe_professors/sources/`.

This would look similar to:

```
.
|-- autoload.php
|-- core
|-- index.php
|-- modules
|   |-- contrib
|   |   |-- address
|   |   |-- entity_reference_revisions
|   |   |-- migrate_plus
|   |   |-- migrate_source_csv
|   |   |-- migrate_tools
|   |   `-- paragraphs
|   `-- custom
|       `-- mbe_professors
|           |-- composer.json
|           |-- config
|           |-- mbe_professors.info.yml
|           |-- mbe_professors_setup
|           |-- migrations
|           |-- README.md
|           `-- sources
|-- profiles
|-- robots.txt
|-- sites
|-- themes
|-- update.php
`-- web.config
```

Not having the source CSV files in the proper location would trigger and error similar to:

```
[error]  Migration failed with source plugin exception: File path (modules/custom/mbe_professors/sources/mbe_book_paragraph.csv) does not exist.
```

If you want to place the files in a different location, you need to update the path in the corresponding configuration files. That is the `source:path` setting in the migration files.

### Running the migrations

The Migrate Tools module provides drush commands to run the migrations.

```
drush migrate:import mbe_book_paragraph
drush migrate:import mbe_photo_field
drush migrate:import mbe_professors
```

**Note:** The commands above work for Drush 10. In Drush 8 the command names and aliases are different. Execute `drush list --filter=migrate` to verify the proper commands for your version of Drush.

After the migrations are run successfully, visit /professors to see the content that was imported.

### Gotcha

There is an issue when rolling back a migration that depends on another one that interacts with entity revisions. This affects paragraphs migrations.

For this particular demo, if the mbe_professors is rolled back and later imported again, the book paragraphs will not be associated with the nodes.

To fix this, the dependent migration mbe_book_paragraph also needs to be rolled back and migrated again. This can be done via the UI or with the following Drush commands:

```
drush migrate:rollback mbe_professors
drush migrate:rollback mbe_book_paragraph
drush migrate:import mbe_book_paragraph
drush migrate:import mbe_professors
```

**Note:** The commands above work for Drush 10. In Drush 8 the command names and aliases are different. Execute `drush list --filter=migrate` to verify the proper commands for your version of Drush.
