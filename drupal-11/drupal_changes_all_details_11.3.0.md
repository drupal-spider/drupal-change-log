# Drupal Change Records for 11.3.0

## [Static calls to overridden entity type will still work if the entity type is overridden another time](https://www.drupal.org/node/3557464)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

If an entity type is overridden using a hook_entity_type_alter to use a different class, it was possible to use statically the entity type or the overridden version:



```php
OriginalClass::loadMultiple(); // Ok.
OverrideClass::loadMultiple(); // Ok.
```

But if another module overrides again the entity type class, then the static calls of the first override would not work:



```php
Original::loadMultiple(); // Ok.
Override::loadMultiple(); // Error.
AnotherOverride::loadMultiple(); // Ok.
```

Now it is possible to override an entity type class multiple times without issue in the entity type manager finding the correct class to load.


A new `getDecoratedClasses` method had been added in case your code needs to check for an entity type class.


Before:



```php
$entity_type->getOriginalClass() == $class_name
```

After:



```php
in_array($class_name, $entity_type->getDecoratedClasses(), TRUE)
```

The new method allows to support any number of overrides.

---

## [HTMX requests may be configured to use the drupal_htmx wrapper format](https://www.drupal.org/node/3558614)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new `onlyMainContent()` method is available in `Drupal\Core\Htmx\Htmx`. This method adds a data attribute which causes the HTMX request to use the `drupal_htmx` wrapper format paramter.  When this parameter is present, Drupal only returns the main content, excluding header, footers, etc. 


Return the full page with all regions and block on the page.



```php
(new Htmx())
      ->post($form_url)
      ->select('[data-export-wrapper]')
      ->target('[data-export-wrapper]')
```

Only return the main content without any extra data from the blocks and regions around the content.



```php
(new Htmx())
      ->post($form_url)
      ->onlyMainContent()
      ->select('[data-export-wrapper]')
      ->target('[data-export-wrapper]')
```

When the new method is used, a special `data-hx-drupal-only-main-content` data attribute will be added. When this attribute is present on an HTMX enhanced element, the query parameter for the `drupal_htmx` wrapper format created in [#3522597: Return only main content for selected htmx requests](https://www.drupal.org/project/drupal/issues/3522597) will be added to HTMX requests prepared by Drupal.

---

## [Form API callbacks now support callables supported by the CallableResolver](https://www.drupal.org/node/3548821)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Form callbacks such as validate or submit callbacks now support any callables that `Drupal\Core\Utility\CallableResolver` supports.


These callbacks can now live in services and will support dependency injection.


Note that the callback no longer needs to be static.


Before:



```php
$form['#submit'][] = [FileFormHooks::class, 'settingsSubmit'];
...

public static function settingsSubmit(array &$form, FormStateInterface $form_state): void {
```

After:



```php
$form['#submit'][] = static::class . ':settingsSubmit';
...

public function settingsSubmit(array &$form, FormStateInterface $form_state): void {
```

In this example, the callable string is `Drupal\file\Hook\FileFormHooks:settingsSubmit` where `Drupal\file\Hook\FileFormHooks` is a registered service name.


Note


It first checks if the service container is available and uses the callable resolver service to get the callable from the definition. Otherwise, it returns the definition as is.


⚠️ Warning
Using `$this` as a pointer for form callback `[$this, 'someMethod']` is an antipattern and should be avoided as the form will be serialized.

---

## [CSS reset added to Navigation module's toolbar and top bar](https://www.drupal.org/node/3560492)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A CSS reset has been added to the Navigation module to prevent front-end theme styles from leaking into the administrative toolbar and top bar. This reset uses the `:is()` pseudo-class to pair the toolbar selector with a non-existent ID, giving the entire selector the specificity of an ID. 


Inside the new ruleset, the `all: revert;`declaration resets all descendant elements back to the browser’s default styles. This ensures the toolbar renders consistently regardless of the front-end theme. 



```php
:is(#extra-specificity-hack, [data-drupal-admin-styles]) {
  box-sizing: border-box;

  *:not(:where(svg, svg *)) {
    all: revert;
    box-sizing: border-box;

    button {
      font-family: inherit;
    }

    &::after,
    &::before {
      all: revert;
      box-sizing: border-box;
    }
  }
}
```
If a module or theme includes custom styling that targets the top bar, those selectors will need to be updated to use the new, higher-specificity selector. 


Before

```php
[data-drupal-admin-styles] {
  .my-button {
    /* Styles */
  }
}
```
After

```php
:is(#extra-specificity-hack, [data-drupal-admin-styles]) {
  .my-button {
    /* Styles */
  }
}
```

---

## [Migrate process plugins for legacy upgrade are deprecated](https://www.drupal.org/node/3533560)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The following migration process plugins are deprecated and will be removed in Drupal 12.0. There is no replacement.  These plugins are designed specifically for migration from legacy Drupal sites.


Deprecated

- core/modules/block/src/Plugin/migrate/process/BlockPluginId.php

- core/modules/block/src/Plugin/migrate/process/BlockRegion.php

- core/modules/block/src/Plugin/migrate/process/BlockSettings.php

- core/modules/block/src/Plugin/migrate/process/BlockTheme.php

- core/modules/block/src/Plugin/migrate/process/BlockVisibility.php

- core/modules/block/src/Plugin/migrate/process/RolesLookup.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldFormatterSettingsDefaults.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldInstanceDefaults.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldInstanceOptionTranslation.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldInstanceSettings.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldInstanceWidgetSettings.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldOptionTranslation.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldSettings.php

- core/modules/field/src/Plugin/migrate/process/d6/FieldTypeDefaults.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldBundle.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldInstanceDefaults.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldInstanceOptionTranslation.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldInstanceSettings.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldOptionTranslation.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldSettings.php

- core/modules/field/src/Plugin/migrate/process/d7/FieldTypeDefaults.php

- core/modules/file/src/Plugin/migrate/process/d6/FieldFile.php

- core/modules/file/src/Plugin/migrate/process/d6/FileUri.php

- core/modules/filter/src/Plugin/migrate/process/FilterID.php

- core/modules/filter/src/Plugin/migrate/process/FilterSettings.php

- core/modules/filter/src/Plugin/migrate/process/d6/FilterFormatPermission.php

- core/modules/image/src/Plugin/migrate/process/d6/ImageCacheActions.php

- core/modules/language/src/Plugin/migrate/process/ContentTranslationEnabledSetting.php

- core/modules/language/src/Plugin/migrate/process/LanguageDomains.php

- core/modules/language/src/Plugin/migrate/process/LanguageNegotiation.php

- core/modules/language/src/Plugin/migrate/process/LanguageTypes.php

- core/modules/link/src/Plugin/migrate/process/FieldLink.php

- core/modules/node/src/Plugin/migrate/process/d6/NodeUpdate7008.php

- core/modules/path/src/Plugin/migrate/process/PathSetTranslated.php

- core/modules/responsive_image/src/Plugin/migrate/process/ImageStyleMappings.php

- core/modules/search/src/Plugin/migrate/process/SearchConfigurationRankings.php

- core/modules/system/src/Plugin/migrate/process/d6/SystemUpdate7000.php

- core/modules/system/src/Plugin/migrate/process/d6/TimeZone.php

- core/modules/taxonomy/src/Plugin/migrate/process/TargetBundle.php

- core/modules/user/src/Plugin/migrate/process/ConvertTokens.php

- core/modules/user/src/Plugin/migrate/process/ProfileFieldSettings.php

- core/modules/user/src/Plugin/migrate/process/UserLangcode.php

- core/modules/user/src/Plugin/migrate/process/UserUpdate8002.php

- core/modules/user/src/Plugin/migrate/process/d6/ProfileFieldOptionTranslation.php

- core/modules/user/src/Plugin/migrate/process/d6/UserUpdate7002.php

---

## [ImageStyle::getReplacementID is deprecated](https://www.drupal.org/node/3520914)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`\Drupal\image\Entity\ImageStyle::getReplacementID()` is deprecated in Drupal 11.3.0 and will be removed in Drupal 12.0.0. 


There appears to be no usage of the method so no direct replacement for this method is provided. If needed call `getReplacementId()` on the entity storage class.


Before:



```php
$replacement_id = $image_style->getReplacementId();
```

After:



```php
$storage = \Drupal::entityTypeManager()->getStorage($image_style->getEntityTypeId());
$replacement_id = $storage->getReplacementId($image_style->id());
```

---

## [MemoryBackend::garbageCollection() now removes invalid items from memory](https://www.drupal.org/node/3559908)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Executing `MemoryBackend::garbageCollection()` will properly clear invalidated items, freeing up memory.


Previously, invalidating items in the `MemoryBackend` cache would not result in their removal.  This could lead to situations where the size of the cache in RAM would grow without a means to clear it.  It happened because `MemoryBackend::garbageCollection()` was an empty function.

---

## [content_translation_field_sync_widget has been deprecated](https://www.drupal.org/node/3548573)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`content_translation_field_sync_widget` has been deprecated.


`Use Drupal\content_translation\FieldSyncWidget::widget()` instead.


`core/modules/content_translation/content_translation.admin.inc` has been deprecated as well, all functions have been moved to the corresponding hook except `content_translation_field_sync_widget` which has the new service: `FieldSyncWidget`.

---

## [Block content entity reference fields now use the BlockContentSelection plugin by default](https://www.drupal.org/node/3521459)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new EntityReferenceSelection plugin has been added for block_content entities. This automatically filters out non-reusable block_content entities.


This filtering was previously done by `BlockContentHooks::queryEntityReferenceAlter` to automatically add the condition on the reusable field if it didn't already exist.


This automatic addition of the reusable field is now deprecated. You can either:



- Add the condition manually to your custom EntityReferenceSelection plugin.

- Or, extend `\Drupal\block_content\Plugin\EntityReferenceSelection\BlockContentSelection` from your plugin.

---

## [doctrine/annotations has been forked into core](https://www.drupal.org/node/3551049)

- **Version**: 11.3.0, 10.6.0
- **Branch**: 11.x, 10.6.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `doctrine/annotations` package is marked as abandoned upstream. Drupal still uses some of the code from this package to parse PHP annotations and support for this will be not dropped until Drupal 13: [Not providing an attribute class for a plugin that that uses annotation based discovery is now deprecated](https://www.drupal.org/node/3522776). As a result Drupal core has copied the relevant parts of the code from Doctrine and moved them into core.


Code that relied on the `Doctrine\Common\Annotations\AnnotationException` exception should switch to using `Drupal\Component\Annotation\Doctrine\AnnotationException` instead; a backward compatibility layer is provided for this.


Code that relied on other parts of `doctrine/annotations` may choose to add an explicit dependency on the abandoned package, or use the classes that were copied to core (with the caveat that these will be removed in Drupal 13), or rework their code to no longer rely on PHP annotations.

---

## [New noUi property allowing page builders to exclude SDCs](https://www.drupal.org/node/3542594)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

A new noUi property has been added to Single-Directory Components (SDC) YAML schema definitions, to allow exclusions of specific SDC's from page builders (e.g. [Drupal Canvas](https://www.drupal.org/project/canvas) or [Display Builder](https://www.drupal.org/project/display_builder)). Take into account that respecting this flag is each page builder choice.


Details:



- Location: Applicable to *.component.yml files for SDCs.

- Default behavior: noUi default to false (visible in UI).


Usage Example:



```php
Component excluded from the UI

name: Internal Component
description: 'Internal Only'
noUi: true

========================================

Component visible in the UI (default behavior)

name: Public Component
description: 'Public Only'
# noUi omitted, or explicitly set to false
# noUi: false
```

---

## [module:// and theme:// stream wrappers added to core](https://www.drupal.org/node/2352923)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`module://` and `theme://` stream wrappers have been added to Drupal core to make it easier to reference certain files in modules or themes.


Important! When the PHP code needs to refer, include or parse files inside the module or theme directory space, will simply use `_DIR_` instead of these new stream wrappers as this is sufficient, intuitive and more performant.


New stream wrappers
Introduce extension stream wrappers:




Scheme
Description


`module://{name}`
Points to files in the module `{name}` directory. Only enabled modules can be referred.


`theme://{name}`
Points to files in the theme `{name}` directory. Only installed themes can be referred.


Examples
Assuming the `node` module is enabled but `color` module is not:



- Referring to a file:

- Drupal < 11.3: `\Drupal::service('extension.path.resolver')->getPath('module', 'node') . '/test.json'`

- Drupal >= 11.3: `module://node/test.json`




- Referring a resource in a uninstalled or inexistent module:

- Drupal < 11.3:`\Drupal::service('extension.path.resolver')->getPath('module', 'color') . '/test.json'` will return `'/test.json'` and issue a PHP warning - `The following module is missing from the file system: color`

- Drupal >= 11.3: `module://color/test.json` throws an exception





This works identically for themes, except those use the `theme://` scheme instead of `module://`.


For the purposes of security hardening, these stream wrappers can only access files with certain extensions. By default, that list is limited to `.json` files. The list of allowed file extensions is controlled by the `stream_wrapper.allowed_file_extensions` container parameter and cannot be configured or overridden in the UI.

---

## [FiberResumeType enum introduced to allow fiber suspensions to indicate the intent](https://www.drupal.org/node/3556785)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Drupal core started using fibers not to asynchronously wait on external resources but attempt to group up expensive operations such as loading entities and and path alias lookups.


However, a large chunk of rendering for now still happens in a single, non-concurrent fiber. This fiber implementation in \Drupal\Core\Render\Renderer::executeInRenderContext() is there to ensure that the render context is fiber-safe. Until now, that fiber waited a short amount of time on suspend to avoid spin-locking.


Large pages that render and load dozens of entities result in an large amount of fiber suspensions, and those calls to usleep() add up and result in a considerably overhead.


The new enum allows suspensions that do not need to wait for an asynchronous event to indicate that a fiber may be resumed immediately if there is no other fiber that may be executed first instead. This needs to be done explicitly and supported both by code that suspends fiber and the code that manages the fiber.


Indicate that an immediate resume is safe

```php
$fiber->suspend(FiberResumeType::Immediate);
```
Check if a fiber is safe to resume immediately

```php
$resume_type = NULL;
    $fiber = new \Fiber(static fn () => $callable());
    $fiber->start();
    $resume_type = NULL;
    while (!$fiber->isTerminated()) {
      if ($fiber->isSuspended()) {
        $resume_type = $fiber->resume();
      }
      // If the fiber has been suspended and has not signaled that it can be
      // immediately resumed, assume that the fiber is waiting on an async
      // operation and wait a bit.
      if (!$fiber->isTerminated() && $resume_type !== FiberResumeType::Immediate) {
        usleep(500);
      }
```

Fibers that wait for API requests, database queries and similar to not need to change, managing multiple fibers generally does not need to consider this and manual fiber management of a single fiber is very rare and generally not needed. Fiber management will likely be handed of to a revolt event loop in the future: [#3394423: Adopt the Revolt event loop for async task orchestration](https://www.drupal.org/project/drupal/issues/3394423)

---

## [Accessing $this->container from functional tests is deprecated](https://www.drupal.org/node/3492500)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Accessing `$this->container` from Functional tests is deprecated. 


Use `\Drupal::service()` or `\Drupal::getContainer()` instead

---

## [Entity Type definitions can now optionally provide a "link_target" handler](https://www.drupal.org/node/3350853)

- **Version**: 11.3.0
- **Branch**: 11.3,x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The web is full of links. When creating content in Drupal, content creators often want to link to other entities in Drupal. Inside of Drupal's assistive text editor (CKEditor 4 previously, now 5) this has historically been a laborious manual process, with brittle results (risk of typos, copy/paste errors, link rot, and more).


To facilitate creating content that consistently links to entities of all types (`Node`,  but also `File` and `Media`, and even `MenuLinkContent`), a new optional entity handler is introduced: the `link_target` handler:



```php
*     "link_target" = {
 *       "view" = "\Drupal\menu_link_content\Entity\MenuLinkContentLinkTarget",
 *     },
```
(for an entity type that is not downloadable)



```php
*   handlers = {
 *     "storage" = "Drupal\media\MediaStorage",
…
 *     "link_target" = {
 *       "view" = "\Drupal\media\Entity\MediaLinkTargetStandaloneWhenAvailable",
 *       "download" = "\Drupal\media\Entity\MediaLinkTarget",
 *     },
```
(for an entity type that is downloadable)


Such handlers must implement `\Drupal\Core\Entity\EntityLinkTargetInterface`, which has a single method:



```php
/**
   * Gets the generated URL object for a linked entity's link target.
   *
   * @param \Drupal\Core\Entity\EntityInterface $entity
   *   A linked entity.
   *
   * @return \Drupal\Core\GeneratedUrl
   *   The generated URL plus cacheability metadata.
   */
  public function getLinkTarget(EntityInterface $entity): GeneratedUrl;
```

This allows links to a `File` entity (or to a `Media` entity using the `File` source plugin) to automatically be resolved to the underlying file, links to `Media` using the `OEmbed` source plugin to automatically be resolved to the originally embedded URL, and so on.


For example, `FileLinkTarget`:



```php
public function getLinkTarget(EntityInterface $entity): GeneratedUrl {
    assert($entity instanceof FileInterface);
    $url = $entity->createFileUrl(TRUE);
    // The $url is a string, which provides no cacheability metadata.
    assert(is_string($url));
    return (new GeneratedUrl())
      ->setGeneratedUrl($url)
      // No path & route processing means permanent cacheability.
      ->setCacheMaxAge(Cache::PERMANENT);
  }
```
or `MenuLinkContentLinkTarget`:



```php
public function getLinkTarget(EntityInterface $entity): GeneratedUrl {
    assert($entity instanceof MenuLinkContentInterface);
    return $entity->getUrlObject()->toString(TRUE);
  }
```

See [the Change Record for the new Entity Links filter plugin](https://www.drupal.org/node/3524296) for where this is used.

---

## [A new Entity Links Filter format and CKEditor 5 plugin has been added](https://www.drupal.org/node/3524296)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

An exciting new feature has been added to core, if you've used the [Linkit](https://drupal.org/project/linkit) module this will look very familiar.





To enable the Entity Links filter, edit your Text format filters that uses a CKEditor 5 editor and enable the Entity Links filter.


 


By default, only Node entities are exposed as link suggestions. 


Other entity types can be enabled via `hook_entity_bundle_info_alter`.


For example, you could enable link suggestions for all media and taxonomy bundles like so:



```php
<?php

declare(strict_types=1);

namespace Drupal\my_module\Hook;

use Drupal\Core\Hook\Attribute\Hook;

/**
 * Hook implementations for my_module.
 */
class MyModuleHooks {

  /**
   * Implements hook_entity_bundle_info_alter().
   */
  #[Hook('entity_bundle_info_alter')]
  public function entityBundleInfoAlter(array &$bundles): void {
    if (isset($bundles['taxonomy_term'])) {
      foreach ($bundles['taxonomy_term'] as $key => $bundle) {
        $bundles['taxonomy_term'][$key]['ckeditor5_link_suggestions'] = TRUE;
      }
    }

    if (isset($bundles['media'])) {
      foreach ($bundles['media'] as $key => $bundle) {
        $bundles['media'][$key]['ckeditor5_link_suggestions'] = TRUE;
      }
    }
  }

}
```

---

## [New Sequentially constraint added to core](https://www.drupal.org/node/3521594)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

New Sequentially constraint added to core. This constraint allows you to validate sequentially. Showing the first error that shows up instead of all the errors on the data.


It has been applied to `block.schema.yaml` in this issue.



```php
block.block.*:
  mapping:
    region:
      type: string
      label: 'Region'
      constraints:
        NotBlank: []
        Callback:
          callback: ['\Drupal\block\Entity\Block', validateRegion]
```

Now is:



```php
block.block.*:
  mapping:
    region:
      type: string
      label: 'Region'
      constraints:
        Sequentially:
          - NotBlank: []
          - Callback:
              callback: ['\Drupal\block\Entity\Block', validateRegion]
```

---

## [AtLeastOneOfConstraintValidator has been replaced by the default Symfony implementation](https://www.drupal.org/node/3558133)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

By optimizing how we handle Composite constraits from Symfony we no longer need `AtLeastOneOfConstraintValidator` that was introduced in 11.2.0 but moved to the default implementation from Symfony.


If you are using or extending the `AtLeastOneOfConstraint` this should not affect you. Only if you have custom validation logic in the extended class.

---

## [\Drupal\Core\Validation\CompositeConstraintInterface added to bridge Symfony's Composite constraints to Drupal](https://www.drupal.org/node/3558184)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

When implementing composite constraints in Drupal, the constraint class must implement `\Drupal\Core\Validation\CompositeConstraintInterface`. The public static method `\Drupal\Core\Validation\CompositeConstraintInterface::getCompositeOptionStatic()` is a copy of Symfony's `\Symfony\Component\Validator\Constraints\Composite::getCompositeOption()` method and allows the Drupal ConstraintManager to instantiate the nested constraints.

---

## [New TwigAllowed method attribute](https://www.drupal.org/node/3551699)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The access of Twig templates to object methods can now allowed by the TwigAllowed attribute.
Example:



```php
class AttributeAllowTestClass {

  #[\Drupal\Core\Template\Attribute\TwigAllowed]
  public function allowed(): string {
    return __METHOD__;
  }

  public function notAllowed(): string {
    return __METHOD__;
  }

  // Still allowed, but soon deprecated.
  public function getFoo(): string {
    return 'foo';
  }

}
```
Note that



- The legacy behavior of allowing [methods with magic prefixes](https://www.drupal.org/node/2595803) is still working, but may be deprecated in the future.

- The [legacy method of adjusting said patterns via settings.php](https://www.drupal.org/node/2595803) is still working, but discouraged.

---

## [ArchiverManager and other archive management code is deprecated](https://www.drupal.org/node/3556927)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Drupal has a plugin manager and related code to handle listing and extracting archived files. Since the ability to update code via tar.gz or zip file was previously deprecated and removed, there is no use case for generic archive management in core any longer.


The following classes are deprecated for removal in Drupal 12:



```php
\Drupal\Core\Archiver\ArchiverException
\Drupal\Core\Archiver\ArchiverInterface
\Drupal\Core\Archiver\ArchiverManager
\Drupal\Core\Archiver\Tar
\Drupal\Core\Archiver\Zip
```

`\Drupal\Core\Archiver\ArchiveTar`, which is a thin wrapper around PEAR's Archive_Tar, will remain as the bulk config UI import/export interface still uses it.


There is no replacement. For tar.gz files you can switch to using ArchiveTar. For zip files you can use the PHP Zip extension directly.

---

## [CommentManagerInterface::getCountNewComments is deprecated](https://www.drupal.org/node/3551729)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

`\Drupal\comment\CommentManagerInterface::getCountNewComments()` is deprecated, use `\Drupal\history\HistoryManager::getCountNewComments()` instead.


The `comment-count-new` token has moved from the Comment to the History module. To use this token the History module must be installed.

---

## [Calls to \Drupal\Core\Field\FieldStorageDefinitionInterface::getPropertyDefinition() will trigger a deprecation if $name is not a string](https://www.drupal.org/node/3557373)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Calling implementations of` \Drupal\Core\Field\FieldStorageDefinitionInterface::getPropertyDefinition()` with `$name` as a NULL will trigger a deprecation. 


Calling code should be refactor so the `$name` parameter is not NULL.

---

## [Classes used in entity form handlers must implement Drupal\Core\Entity\EntityFormInterface](https://www.drupal.org/node/3528495)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Form handlers that are defined in an entity type's definition and used in route definitions with the `_entity_form` default must implement `Drupal\Core\Entity\EntityFormInterface`.


Previously, if a form handler was used in an `_entity_form` route definition and didn't extend `EntityForm`, an obscure error was reported because `Drupal\Core\Entity\EntityTypeManager::getFormObject` calls several methods on the form object defined by `EntityFormInterface`.


It is recommended to extend `\Drupal\Core\Entity\EntityForm` in your form handler classes.

---

## [Migrate Drupal UI is deprecated](https://www.drupal.org/node/3533901)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The Migrate Drupal UI module is deprecated in Drupal 11.3.0 and will be removed in Drupal 12.0.0.  There is no replacement.


If you want to keep using the functionality provided by the Migrate Drupal UI module, read the [recommendations for Migrate Drupal UI](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-migrate-drupal-ui).

---

## [Field Layout module is deprecated](https://www.drupal.org/node/3557095)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Field Layout module has been deprecated.


If you want to keep using the functionality provided by Field Layout, read the [recommendations for Field Layout](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-field-layout).

---

## ["Created" fields are excluded from default content by default](https://www.drupal.org/node/3557689)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Default content is now exported without the `created` field by default. The reasoning is that it's confusing, upon import, to see content that was created at an arbitrary time, rather than the time of import (which, as far as Drupal is concerned, is the time the content was created).


If you want to include created times in default content, you can do this with an event subscriber that marks the fields as included:



```php
use Drupal\Core\DefaultContent\PreExportEvent;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

/**
 * Subscribes to default content-related events.
 *
 * @internal
 *   Event subscribers are internal.
 */
class DefaultContentSubscriber implements EventSubscriberInterface {

  /**
   * {@inheritdoc}
   */
  public static function getSubscribedEvents(): array {
    return [PreExportEvent::class => 'preExport'];
  }

  /**
   * Reacts before an entity is exported.
   *
   * @param \Drupal\Core\DefaultContent\PreExportEvent $event
   *   The event object.
   */
  public function preExport(PreExportEvent $event): void {
    // To automatically export all `created` fields:
    foreach ($event->entity->getFieldDefinitions() as $field_definition) {
      if ($field_definition->getType() === 'created') {
        $event->setExportable($field_definition->getName());
      }
    }

    // To only export a specific one:
    if ($event->entity->getEntityTypeId() === 'node') {
      $event->setExportable('created');
    }
  }

}
```

---

## [\Drupal\migrate\Plugin\migrate\process\StaticMap::transform() cannot map NULL values unless there is a default value or bypass is set](https://www.drupal.org/node/3557003)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Currently the static map plugin will map NULL to values using an empty string array key. For example the following StaticMap configuration



```php
map:
  '': 'Mapped null value'
  this: that
```
will map NULL values to `'Mapped null value'`. This will trigger a deprecation in Drupal 11.3.0. To avoid the deprecation the migration can either use the `default_value` or `bypass` setting. Setting `bypass` to TRUE will return NULLs for NULL values. Setting a `default_value` will return the default value for NULL values.

---

## [Do not call \Drupal\Core\Entity\EntityTypeBundleInfo::getBundleInfo() with a NULL value](https://www.drupal.org/node/3557136)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Calling `Drupal\Core\Entity\EntityTypeBundleInfo::getBundleInfo()` with a non-string entity type ID, such as NULL, is deprecated.  In Drupal 12, an exception will be thrown.


It is recommended to change the calling code to avoid passing a non-string entity type ID.

---

## [Loading revisions now uses the static and persistent cache like](https://www.drupal.org/node/3553211)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Loading revisions now uses the cache if the entity type is configured to do so.


This has the following implications for calls to load(), loadMultiple(), loadRevision() and loadRevisionMultiple():



- 
Better performance
Loading the same revision multiple times during the same request returns the same object from the static cache and on following requests, will return it from the persistent cache, resulting in fewer queries being executed.


The following is now true, which was a not the case before.



```php
$storage = \Drupal::entityTypeManager()->getStorage('node');
assert($storage->loadRevision(1) === $storage->loadRevision(1));
```


- 
Loading an entity by ID also puts it in the revision static cache (but not the persistent cache)
Loading a revisionable entity by ID also makes it available in the static cache when loading it by its revision ID. This is not true for the persistent cache due to the performance cost of writing to the persistent cache (cache tags).


Because the revision is then already in the static cache, calling loadRevision() will also not put that revision into the persistent cache on the same request.


The following is now true, which was the not the case before.



```php
$storage = \Drupal::entityTypeManager()->getStorage('node');
$node = $storage->load(1);
assert($node === $storage->loadRevision($node->getRevisionId()));
```


- 
Loading a revision does not put the entity into the regular static cache
The reverse is not true. Loading a revision does not put into the static cache for the entity ID, since there are cases where a different revision than the default is loaded, such as workspaces:



```php
$storage = \Drupal::entityTypeManager()->getStorage('node');
$revision = $storage->loadRevision(1);
assert($revision === $storage->load($revision->id()));
```



Subclasses of ContentEntityStorageBase may need some adjustments to benefit from this, implementations of ContentEntityStorageInterface that do not extend from ContentEntityStorageBase must implement loadRevisionUnchanged() now.

---

## [The workspaces.association service has been replaced by workspaces.tracker](https://www.drupal.org/node/3551450)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `workspaces.association` service has been deprecated, and a new `workspaces.tracker` has been added.


  Here's an overview of the deprecated and new interfaces in Drupal 11.3.0:





WorkspaceAssociationInterface (deprecated)
WorkspaceTrackerInterface (replacement)




`trackEntity(RevisionableInterface $entity, WorkspaceInterface $workspace)`
`trackEntity(string $workspace_id, RevisionableInterface $entity): void`


`workspaceInsert(WorkspaceInterface $workspace)`
Removed


`getTrackedEntities($workspace_id, $entity_type_id = NULL, $entity_ids = NULL)`
`getTrackedEntities(string $workspace_id, ?string $entity_type_id = NULL, ?array $entity_ids = NULL): array`


`getTrackedEntitiesForListing($workspace_id, ?int $pager_id = NULL, int|false $limit = 50): array`
`getTrackedEntitiesForListing(string $workspace_id, ?int $pager_id = NULL, int|false $limit = 50): array`


`getAssociatedRevisions($workspace_id, $entity_type_id, $entity_ids = NULL)`
`getAllTrackedRevisions(string $workspace_id, string $entity_type_id, ?array $entity_ids = NULL): array`


`getAssociatedInitialRevisions(string $workspace_id, string $entity_type_id, array $entity_ids = [])`
`getTrackedInitialRevisions(string $workspace_id, string $entity_type_id, ?array $entity_ids = NULL): array`


`getEntityTrackingWorkspaceIds(RevisionableInterface $entity, bool $latest_revision = FALSE)`
`getEntityTrackingWorkspaceIds(RevisionableInterface $entity, bool $latest_revision = FALSE): array`


`moveTrackedEntities(string $source_workspace_id, string $target_workspace_id, ?string $entity_type_id = NULL, ?array $entity_ids = NULL): void`
`moveTrackedEntities(string $source_workspace_id, string $target_workspace_id, ?string $entity_type_id = NULL, ?array $entity_ids = NULL): void`


`deleteAssociations($workspace_id = NULL, $entity_type_id = NULL, $entity_ids = NULL, $revision_ids = NULL)`
`deleteTrackedEntities(?string $workspace_id = NULL, ?string $entity_type_id = NULL, ?array $entity_ids = NULL, ?array $revision_ids = NULL): void`


`initializeWorkspace(WorkspaceInterface $workspace)`
`initializeWorkspace(WorkspaceInterface $workspace): void`

---

## [Memory management removed from MigrateExecutable](https://www.drupal.org/node/3139212)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Migrate has a long history of reclaiming memory via PHP garbage collection. As entities get added, this memory reclamation gets kicked off.


Since [https://www.drupal.org/project/drupal/issues/3498154](https://www.drupal.org/project/drupal/issues/3498154) the static entity cache is now implemented using an LRU memory cache, which cycles items out of memory when it gets full. This signficantly improves the memory usage of long-running migrations without requiring migrate's own memory management to run at all. Therefore the memory management logic in MigrateExecutable has been removed.


If sites are on hosting with a very low memory limit, it is possible to manually tweak the number of slots in the entity memory LRU cache, so that maximum PHP memory usage stays at a lower level. See [https://www.drupal.org/node/3512407](https://www.drupal.org/node/3512407)

---

## [Invalid attributes are changed in language switcher block HTML](https://www.drupal.org/node/3556699)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

The default HTML for the language switcher block contains an unordered list whose list items had an `hreflang` attribute which `hreflang` contained the two-letter code for the linked language. The `hreflang` attribute is not allowed on `<li>` elements. This attribute has been changed to `data-drupal-language`.


Before:



```php
<ul class="links">
  <li hreflang="en" data-drupal-link-system-path="&lt;front&gt;"><a href="/" class="language-link" hreflang="en" data-drupal-link-system-path="&lt;front&gt;">English</a></li>
  <li hreflang="is" data-drupal-link-system-path="&lt;front&gt;"><a href="/is" class="language-link" hreflang="is" data-drupal-link-system-path="&lt;front&gt;">Icelandic</a></li>
</ul>
```

After:



```php
<ul class="links">
  <li data-drupal-language="en" data-drupal-link-system-path="&lt;front&gt;"><a href="/" class="language-link" hreflang="en" data-drupal-link-system-path="&lt;front&gt;">English</a></li>
  <li data-drupal-language="is" data-drupal-link-system-path="&lt;front&gt;"><a href="/is" class="language-link" hreflang="is" data-drupal-link-system-path="&lt;front&gt;">Icelandic</a></li>
</ul>
```

---

## [The .engine extension has been deprecated. Use tagged services instead.](https://www.drupal.org/node/3547356)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Theme engine extensions have been deprecated. Theme engines must now be a tagged services in modules.


In a module implement `\Drupal\Core\Theme\ThemeEngineInterface`
Tag the service as a `theme_engine`:



```php
Drupal\Core\Template\TwigThemeEngine:
    class: Drupal\Core\Template\TwigThemeEngine
    autowire: true
    tags:
      - { name: theme_engine, engine_name: twig }
```

If your theme depends on a custom engine you will need to do the following.



- Add a dependency to the module providing the dependency.

- Ensure the module gets installed as part of the conversion otherwise it will not be available


Theme engines no longer support preprocess hooks.
Any preprocess hooks implemented by the theme engine should be moved to the module providing the new theme engine service.


The ThemeManager service now requires the set of theme_engine services to be passed into the constructor as a service locator:



```php
#[AutowireLocator('theme_engine', 'engine_name')]
    protected ContainerInterface $themeEngines,
```

---

## [Method getValuesSetDuringRequest() added to Drupal\Core\State\StateInterface](https://www.drupal.org/node/3519307)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

To provide a way to track state value changes within a request, the `getValuesSetDuringRequest()` method has been added to `Drupal\Core\State\StateInterface`. All classes that implement `StateInterface` but do not extend `Drupal\Core\State\State` will need to implement this new method.


Method signature:



```php
/**
   * Returns the values if a key has been modified during the request.
   *
   * @param string $key
   *   The key to get the values for.
   *
   * @return array{value: mixed, original: mixed}|null
   *   An array containing:
   *     - value: The last value set during the request.
   *     - original: The initial value at the start of the request.
   *   If the key was not set, NULL is returned.
   */
  public function getValuesSetDuringRequest(string $key): ?array;
```

An example of how this method is used is in `Drupal\Core\EventSubscriber\MaintenanceModeSubscriber`, which now can log when maintenance mode has been enabled or disabled.



```php
/**
   * Logs changes to maintenance mode.
   *
   * @param \Symfony\Component\HttpKernel\Event\TerminateEvent $event
   *   The event object.
   */
  public function onTerminate(TerminateEvent $event): void {
    $values = $this->state->getValuesSetDuringRequest('system.maintenance_mode');
    if ($values && $values['original'] !== $values['value']) {
      if ($values['value']) {
        $this->logger->info('Maintenance mode enabled.');
      }
      else {
        $this->logger->info('Maintenance mode disabled.');
      }
    }
  }
```

---

## [New ConfigImporterFactory service](https://www.drupal.org/node/3394638)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new service, `config.importer.factory` has been added which allows creation of a ConfigImporter object.


This should be used rather than instantiating the ConfigImporter class directly.


Before:



```php
$config_importer = new ConfigImporter(
  $storage_comparer,
  \Drupal::service('event_dispatcher'),
  \Drupal::service('config.manager'),
  \Drupal::service('lock.persistent'),
  \Drupal::service('config.typed'),
  \Drupal::service('module_handler'),
  \Drupal::service('module_installer'),
  \Drupal::service('theme_handler'),
  \Drupal::service('string_translation'),
  \Drupal::service('extension.list.module'),
  \Drupal::service('extension.list.theme')
);
```

After:



```php
$config_importer = \Drupal::service('config.importer.factory')->get($storage_comparer);
```

---

## [node.add and node.add_page routes have new aliases](https://www.drupal.org/node/3555319)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

As part of standardising Node route generation, `\Drupal\node\Entity\NodeRouteProvider` now extends `\Drupal\Core\Entity\Routing\DefaultHtmlRouteProvider\DefaultHtmlRouteProvider`. 


To maintain backwards compatibility, the `node.add` and `node.add_page` routes still exist, and the forwards compatible routes are now aliases of them.


The `entity.node.add_form` route is now an alias of `node.add`.


The `entity.node.add_page` route is now an alias of `node.add_page`.


In future versions of Drupal, `node.add` and `node.add_page` will be deprecated.

---

## [Route option added for routes designed to serve HTMX requests](https://www.drupal.org/node/3547745)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new route option `_htmx_route` is added.  When this option is `TRUE`, the HTMX renderer created in [#3522597: Return only main content for selected htmx requests](https://www.drupal.org/project/drupal/issues/3522597) will be used to return a response that contains only the main content.


Example



```php
test_htmx.attachments.route_option:
  path: '/htmx-test-attachments/route-option'
  defaults:
    _title: 'Using _htmx_route option'
    _controller: '\Drupal\test_htmx\Controller\HtmxTestAttachmentsController::replace'
  requirements:
    _permission: 'access content'
  options:
    _htmx_route: TRUE
```

---

## [file_managed_file_submit() is deprecated](https://www.drupal.org/node/3534091)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`file_managed_file_submit() `is deprecated and moved to `\Drupal\file\Element\ManagedFile::submit()`

---

## [ModuleHandler addProfile and addModule no longer do anything.](https://www.drupal.org/node/3550193)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

ModuleHandler::addProfile and ModuleHandler:addModule are scheduled for removal in drupal 12.


Since the deprecation additional issues with the underlying method ModuleHandler::add have been found.


As a result we have removed add and updated the deprecation messages to reflect that addProfile addModule no longer perform any actions.


[https://www.drupal.org/node/3491200](https://www.drupal.org/node/3491200) is the original Change record.

---

## [ConfigSingleExportForm now has a dynamically updated URL](https://www.drupal.org/node/3546732)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The `ConfigSingleExportForm` has been converted to use HTMX instead of Ajax API.  Although no change will be discernible in the form itself, using HTMX, the url now dynamically updates to reflect the state of the form.


This means that users can use their browser's back button to return to a previous state, or copy and share the url with a colleague to point toward a specific configuration.

---

## [Removed support for PHPUnit 10](https://www.drupal.org/node/3546970)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Framework support for testing with PHPUnit 10 is removed.


Core and contrib can only run tests with PHPUnit 11.


Test classes using annotation metadata still work under PHPUnit 11, but throw PHPUnit deprecations that lead to CI job failures if appropriately set up. It is strongly recommended to convert annotations to attributes as described in [Tests with PHPUnit 10 attributes are now supported](https://www.drupal.org/node/3447698).


Drupal 11 core has completed migration of tests from annotation to attributes in [#3535662: [meta] Convert test metadata from annotations to attributes](https://www.drupal.org/project/drupal/issues/3535662).


Support for annotation metadata will be only completely removed in PHPUnit 12. However, it is possible that preparation issues to support it during Drupal 11 lifecycle may lead to the testing framework stop working properly if annotations are still in use.

---

## [Hooks in themes can now be OOP](https://www.drupal.org/node/3551652)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

Themes now support OOP hooks.
Theme namespaces are now registered in the container.
ThemeInstaller now resets the container in install and uninstall.


Use the same `#[Hook()]` attribute in themes as modules.


Object oriented hooks in themes must be in the hook namespace, i.e. src/Hook/HookClass.php


The hook method must have a `#[Hook()]` attribute.


There are some important differences between hooks in themes and hooks in modules.


The Hook attribute in themes does not support the order or module parameters.
Themes do not support `ReorderHook`.
Themes do not support `RemoveHook`.
The order of hook implementation invocation in themes is fixed, active theme and it's base themes in reverse order.


Themes generally only support a subset of alters and a select few normal hooks.
Supported alter hooks in core: (contrib can support others) check for themeManager->alter and themeManager->alterForTheme



- css_alter

- js_alter

- js_settings_alter

- library_info_alter

- form_alter

- form_BASE_FORM_ID_alter

- form_FORM_ID_alter

- element_info_alter

- page_attachments_alter

- theme_registry_alter

- hook_theme_suggestions_alter

- hook_theme_suggestions_HOOK_alter

- views_ui_display_tab_alter

- views_ui_display_top_alter

- plugin_filter_layout_alter

- plugin_filter_layout__layout_alter



- hook_views_pre_render

- hook_views_post_render

- hook_theme



- hook_preprocess

- hook_preprocess_HOOK


ThemeInstaller last parameter must be the Drupal kernel not the component plugin manager.


ThemeManager now requires keyvalue and cache bootstrap.


The DrupalKernel has deprecated the following methods: 



- getModulesParameter

- getModuleFileNames

- getModuleNamespacesPsr4


Edge cases that are no longer supported:



- Themes cannot implement oop hooks on behalf of other themes.

- Modules cannot implement hooks on behalf of themes.


Backwards compatibility
When a hook has been converted add `#[LegacyHook]` to the procedural implementation.
You MUST maintain the current procedural implementation for themes.
Using Drupal::service('HookService') will not work since this feature also introduced adding the autowired services.


Note
This does not affect module hooks in any way including preprocess and hook_theme.

---

## [Block plugins implementing CacheOptionalInterface will not have their own render cache entries](https://www.drupal.org/node/3554070)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Block plugin classes can implement `Drupal\Core\Cache\CacheOptionalInterface` so that when block entities using those plugins are rendered, those blocks will not have their own entries in the render cache.


While such blocks themselves are not render cached, implementing `CacheOptionalInterface` differs from setting the cache max-age of the block to 0 in that the 0 max-age bubbles to wrapping render elements and prevents the page containing the block from being cached in the dynamic page cache. A `CacheOptionalInterface` block in a page will allow that page to be cached in the dynamic page cache.


If a block plugin implements `CacheOptionalInterface`, and its `createPlaceholder() ` method returns `TRUE`, then when rendered, the block will be placeholdered with the appropriate cache metadata and without an entry in the render cache. Cache contexts for the block will not bubble to the dynamic page cache, and the cached markup in the dynamic page cache entry contains the non-replaced placeholder markup.


For example, the language switcher block (`Drupal\language\Plugin\Block\LanguageBlock`) implements `CacheOptionalInterface`, and its `createPlaceholder() ` method returns `TRUE`. This means that individual pages containing the language switcher block can be cached in the dynamic page cache, and the language switcher block variations by cache contexts (for example, query parameters) for this will not result in multiple dynamic page cache entries for the page.


Note that if a language switcher block or similar is placed in a node layout via layout builder, or otherwise rendered inside another render element that has its own render cache entries, then there could possibly be multiple cache entries for that wrapping render element based on cache contexts bubbled up from the block.

---

## [Kernel tests can use hook attributes to test hooks](https://www.drupal.org/node/3553794)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Kernel tests can declare hook implementations to add hook testing. For example if you want to test hook_node_create() you can add the following method to your kernel test class:



```php
/**
   * Implements hook_ENTITY_TYPE_create() for node entities.
   */
  #[Hook('node_create')]
  public function nodeCreate(): void {
    // Some logic.
  }
```

Note that hooks declared in Kernel tests do not support the hook ordering / removal capabilities.

---

## [Drupal\taxonomy\Form\OverviewTerms now extends Drupal\Core\Entity\EntityForm](https://www.drupal.org/node/3528300)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

To support better extensibility of the Overview form for Taxonomy vocabularies, `Drupal\taxonomy\Form\OverviewTerms` now extends `Drupal\Core\Entity\EntityForm`


The overview route has subsequently been updated as well
From
`$route->setDefault('_form', 'Drupal\taxonomy\Form\OverviewTerms')`
To
`$route->setDefault('_entity_form', 'taxonomy_vocabulary.overview')`


This gives developers an easy way to override this form via `hook_entity_type_alter`


`$entity_types['taxonomy_vocabulary']->setFormClass('overview', 'Drupal\my_module\MyOverviewTerms');`

---

## [theme_get_setting() is deprecated](https://www.drupal.org/node/3035289)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `theme_get_setting()` method is deprecated. It has been replaced by a new `Drupal\Core\Extension\ThemeSettingsProvider` service.


Before

```php
// Get a setting from 'bartik' theme.
$logo = theme_get_setting('logo.url', 'bartik');

// Clear the 'bartik' theme settings static cache.
$theme_settings = &drupal_static('theme_get_setting');
unset($theme_settings[$theme_name]);

// Clear the theme settings static cache for all themes.
drupal_static_reset('theme_get_setting');
```
After

```php
// It's recommended to inject the service where possible. 
$themeSettingsProvider = \Drupal::service('Drupal\Core\Extension\ThemeSettingsProvider');

// Get a setting from 'bartik' theme.
$logo = $themeSettingsProvider->getSetting('logo.url', 'bartik');

// Clear the 'bartik' theme settings static cache.
\Drupal::service('cache.memory')->delete('theme_settings:bartik');

// Clear the theme settings static cache for all themes.
\Drupal::service('cache_tags.invalidator')->invalidateTags(['config:system.theme.global']);
```

Additionally, not passing the `$themeSettingsProvider` service to the  `Drupal\Core\Render\BareHtmlPageRenderer` and `Drupal\system\Plugin\Block\SystemBrandingBlock` constructors is deprecated.


These parameters will be mandatory in Drupal 12.0.0.

---

## [_system_default_theme_features is deprecated](https://www.drupal.org/node/3554127)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`_system_default_theme_features` is deprecated.


Use `\Drupal\Core\Extension\ThemeSettingsProvider::DEFAULT_THEME_FEATURES` instead.

---

## [New WorkspaceSwitchEvent event added](https://www.drupal.org/node/3553871)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new `WorkspaceSwitchEvent` event has been added, which provides the ability to react when the active workspaces changes, either on switching from Live to a workspace or the other way around.


Code example

```php
class WorkspaceSwitchSubscriber implements EventSubscriberInterface {

  public function onWorkspaceSwitch(WorkspaceSwitchEvent $event): void {
    // Clear a custom static cache entry.
  }

  public static function getSubscribedEvents(): array {
    $events[WorkspaceSwitchEvent::class][] = 'onWorkspaceSwitch';

    return $events;
  }

}
```

---

## [WorkspaceManagerInterface::purgeDeletedWorkspacesBatch() has been deprecated](https://www.drupal.org/node/3553582)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`WorkspaceManagerInterface::purgeDeletedWorkspacesBatch()` has been deprecated with no replacement.


It's code has been moved to `\Drupal\workspaces\Hook\WorkspacesHooks::cron()`, which can be called manually if needed.

---

## [New cache.memory cache bin, replaces cache.static, MemoryCacheInterface alias deprecated](https://www.drupal.org/node/3546856)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`cache.memory` is a standardized memory cache bin that can be used to store data in memory, for example as part of a backendchain combined with a persistent cache bin or as an alternative to storing data in a property.


It is backed by the \Drupal\Core\Cache\MemoryCache\MemoryCache backend, which does not serialize data between requests and is therefore faster to use. It does however support in-request cache tag invalidations, which allows to have cached data automatically invalidated when cached in the same request. Cache tag invalidations that happen in separate requests will not affect cache items stored in this bin.


The existing cache bin cache.static is deprecated. It is backed by the \Drupal\Core\Cache\MemoryBackend, which serializes and deserializes cached data to avoid changing it by reference. It is designed to be a drop-in replacement for a persistent cache in Unit or Kernel tests and should not be used for regular runtime operations. The `cache.backend.memory` cache backend is also deprecated. Update cache configurations, either default_backend in a cache bin service or in $settings.php, to use `cache.backend.memory.memory` instead.


The `Drupal\Core\Cache\MemoryCache\MemoryCacheInterface` service is also deprecated, it was incorrectly defined as an alias for the `entity.memory_cache` which should only be used by entity storages to store loaded entities in memory. If your service relied on Autowire to have this cache bin injected, use an `#Autowire` attribute instead.


Before:



```php
public function __contruct(
  MemoryCacheInterface $cache
) {};
```

After:



```php
public function __construct(
  #[Autowire('cache.memory')]
  CacheBackendInterface $cache
) {};
```

Note: As with other generic cache bins, this is shared across all usages, ensure that unique keys are used by prefixing them with the module, class or similar unique prefixes.

---

## [Plugins used in entities with plugin collections can react when the entities' dependencies are removed](https://www.drupal.org/node/3549101)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

When a configurtion entity implementing `EntityWithPluginCollectionInterface` is reacting to dependency removal, it iterates through all the plugins in the plugin collection to check whether any plugin implements `Drupal\Core\Plugin\RemovableDependentPluginInterface`, and if so, invokes the plugin's `onCollectionDependencyRemoval()` method, so that the plugin can react to the dependency removal as well. Implementations of `onCollectionDependencyRemoval()` can be used to update the configuration of the plugin itself and notify the entity to recalculate its dependencies and/or remove the plugin from entity's plugin collection. This can prevent the entity from being unnecessarily deleted on dependency removal if the change to the plugin configuration or the removal of the plugin results in the entity no longer having the dependencies.


`onCollectionDependencyRemoval()` returns a Drupal\Core\Plugin\RemovableDependentPluginReturn enum that has one of three values:



- `RemovableDependentPluginReturn::Changed` if the configuration of the plugin instance has changed

- `RemovableDependentPluginReturn::Remove` if the plugin instance should be removed from the plugin collection

- `RemovableDependentPluginReturn::Unchanged` if the configuration of the plugin instance has not changed.


For example, the `Drupal\image\Entity\ImageStyle` configuration entity type has a collection of `Drupal\image\ImageEffectInterface` plugins. Image effect plugin classes that extend `Drupal\image\ImageEffectBase` have this `onCollectionDependencyRemoval()` implementation that removes the plugin from the entity's collection if the module providing the plugin is uninstalled:



```php
public function onCollectionDependencyRemoval(array $dependencies): RemovableDependentPluginReturn {
    // If the module that provides the image effect plugin is uninstalled,
    // the plugin instance should be removed from the collection.
    return in_array($this->getPluginDefinition()['provider'], $dependencies['module'] ?? []) ? RemovableDependentPluginReturn::Remove : RemovableDependentPluginReturn::Unchanged;
  }
```

---

## [WorkspaceManager::getActiveWorkspace() return value updated](https://www.drupal.org/node/3553092)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The return value of `WorkspaceManager::getActiveWorkspace()` when there is no active workspace was previously undocumented (it returned `FALSE`). This has been changed and documented to return `NULL` instead.


This change was made to enable better use of null-safe operators and type hints. For example, when you need to execute a method on the active workspace, instead of a conditional block using `hasActiveWorkspace()`, the code be simplified to:



```php
$active_workspace_id = $this->workspaceManager->getActiveWorkspace()?->id();
```

---

## [New Workspace Provider system](https://www.drupal.org/node/3553089)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Overview
The workspace provider system introduces a plugin-like architecture based on tagged services that allows different implementations of workspace behavior. This enables modules to create custom workspace types with specialized behavior while keeping standard Drupal workspaces separate in the UI.


Architecture
All workspace providers must implement `Drupal\workspaces\Provider\WorkspaceProviderInterface`, which defines a `getId()` method, custom access control via `checkAccess()`, and entity lifecycle hooks.


`Drupal\workspaces\Provider\WorkspaceProviderBase` provides the default workspace behavior. Custom providers should extend this class and override only the methods they need to customize.


`Drupal\workspaces\Provider\DefaultWorkspaceProvider` is the built-in provider used by standard Drupal workspaces.


How It Works
Entity Operations Delegation
All entity operations now delegate to the active workspace's provider. This means when content is edited in a workspace, the workspace's provider determines the behavior.


UI Filtering
By default, only workspaces using the `default` provider are shown in:



- The workspaces listing page (`/admin/config/workflow/workspaces`)

- Entity reference fields (parent workspace selector)

- The workspace switcher form


Creating a custom provider
1. Define Your Provider Class

```php
namespace Drupal\my_module\Provider;

use Drupal\workspaces\Provider\WorkspaceProviderBase;

class MyCustomProvider extends WorkspaceProviderBase {

  public static function getId(): string {
    return 'my_custom';
  }

  // Override only the methods you need to customize
  public function entityPresave(EntityInterface $entity): void {
    // Your custom workspace behavior
  }
}
```
2. Register Your Provider as a Service
`# my_module.services.yml
services:
  Drupal\my_module\Provider\MyCustomProvider:
    autowire: true
`


3. Create Workspaces with Your Provider

```php
use Drupal\workspaces\Entity\Workspace;

$workspace = Workspace::create([
  'id' => 'my_workspace',
  'label' => 'My Custom Workspace',
  'provider' => 'my_custom',  // Your provider ID
]);
$workspace->save();
```

---

## [Using the 'access callback' key in views definition is deprecated](https://www.drupal.org/node/3539918)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Using the 'access callback' key in the definition of views handlers is now deprecated.


If you need to modify access rules then create a custom handler and override the access method.


Once this is removed the access will return false by default.

---

## [Contact module removed from the Standard profile](https://www.drupal.org/node/3553411)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Contact module is removed from the Standard profile. This is one of the steps to deprecate and remove Contact from core. See [#3520460: [meta] Tasks to deprecate the Contact module](https://www.drupal.org/project/drupal/issues/3520460).


New sites using the Standard profile need to install the module manually.

---

## [Config actions can now be skipped if the entity does not exist](https://www.drupal.org/node/3552215)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Recipes can now skip running config actions on entities that don't exist. For example:



```php
config:
  actions:
    ?canvas.component.block.project_browser_block.recipes:
      disable: []
```

This will run the `disable` config action on the `canvas.component.block.project_browser_block.recipes` config entity -- if and only if that entity exists. If it doesn't exist, all of its config actions will be skipped.


This syntax cannot be combined with wildcards. So this, for example, will throw an exception:



```php
config:
  actions:
    ?canvas.component.block.system_menu_block.*:
      disable: []
```

The reason this won't work with wildcards is because wildcards, by their nature, imply the "skip if not exists" behavior. If a wildcard doesn't turn up any config entities to work on, nothing happens anyway -- so using `?` in combination with that would be redundant.

---

## [template_preprocess_HOOK, and the file and includes keys in hook_theme definitions have been deprecated](https://www.drupal.org/node/3549500)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

template_preprocess_HOOK functions have been deprecated. They are defined as initial preprocess callbacks: see [template_preprocess_HOOK are defined as callbacks in the theme hook](https://www.drupal.org/node/3504125).


The file and includes keys on hook_theme have also been deprecated.


These allowed conditionally loading some files which contained relevant functions for the given definition. The only remaining reason to use them was template_preprocess callbacks. Since these have been moved to initial preprocess callbacks these keys are no longer necessary.


hook_theme() definitions  that still rely on these keys should remove them when converting their template_preprocess functions to initial preprocess and move the code to a class, which will be automatically loaded. See the linked change record for more information.

---

## [Add AJAX command to Views module that sets the browser URL without refreshing the page](https://www.drupal.org/node/3552223)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The views module has an AJAX command that allows code to set the browser URL without reloading the page.



```php
use Drupal\views\Ajax\SetBrowserUrl;

        /** @var \Drupal\Core\Url $url */
        /** @var \Drupal\Core\Ajax\AjaxResponse $response */
        $response->addCommand(new SetBrowserUrl($url->toString()));
```

---

## [New permission available to control the Published status of Nodes](https://www.drupal.org/node/3528500)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

A new permission `administer node published status` has been added to the node module. Granting this permission to a role will allow users in that role to toggle the Published checkbox when editing a node.


Previously, access to this field was locked behind the `administer nodes` permission.

---

## [PluginBase provides create() factory method with autowired parameters](https://www.drupal.org/node/3542837)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Classes that implement `\Drupal\Core\DependencyInjection\ContainerInjectionInterface` must provide a `create()` factory method to instantiate the object.


Now that [services can be autowired](https://www.drupal.org/node/3218156), the `PluginBase` class has been extended with a generic version of the `create()` method suitable for most plugins with dependency injection.


Previously, a constructor and `create()` method would be something like this:



```php
public function __construct(
    array $configuration, $plugin_id,
    $plugin_definition,
    EntityTypeManagerInterface $entity_type_manager,
    BlockManagerInterface $block_manager
  ) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);

    $this->entityTypeManager = $entity_type_manager;
    $this->blockManager = $block_manager;
  }

  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    return new static(
      $configuration,
      $plugin_id,
      $plugin_definition,
      $container->get('entity_type.manager'),
      $container->get('plugin.manager.block')
    );
  }
```

Now it is possible to drop the create() method:



```php
public function __construct(
    array $configuration, $plugin_id,
    $plugin_definition,
    #[Autowire(service: 'entity_type.manager')]
    EntityTypeManagerInterface $entity_type_manager,
    BlockManagerInterface $block_manager
  ) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);

    $this->entityTypeManager = $entity_type_manager;
    $this->blockManager = $block_manager;
  }
```
Make sure the plugin implements `Drupal\Core\Plugin\ContainerFactoryPluginInterface` to proceed with autowiring.


In simple cases, the service can be inferred from the interface name alone. If a service cannot be determined from the interface directly, an exception will be raised, and you can add the #[Autowire] attribute to specify the service ID.

---

## [Access to rebuild node permissions now requires the "rebuild node access permissions" permission](https://www.drupal.org/node/3521446)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Access to rebuild node permissions now requires a new permission `rebuild node access permissions`


Previously it required the `administer nodes` permission.

---

## [New trait assists classes building render arrays for HTMX](https://www.drupal.org/node/3549174)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

HTMX passes information about the context of the request in a [set of custom headers](https://htmx.org/reference/#request_headers).  This information is useful to developers building dynamic responses.


The trait `Drupal\Core\Htmx\HtmxRequestInfoTrait` is included in Drupal 11.3 and has been added to `Drupal\Core\Form\FormBase`.  This trait can be added to a controller or any other class building render arrays for HTMX.


`HtmxRequestInfoTrait` includes an abstract method which must be implemented on the exhibiting class:



```php
/**
   * Gets the request object.
   *
   * @return \Symfony\Component\HttpFoundation\Request
   *   The request object.
   */
  abstract protected function getRequest();
```

Before



```php
$trigger = $this->getRequest()->headers->get('HX-Trigger-Name');
if ($trigger === 'config_type') {
  …
}
elseif ($trigger === 'config_name') {
  …
}
```

After



```php
if ($this->getHtmxTriggerName() === 'config_type') {
  …
}
elseif ($this->getHtmxTriggerName() === 'config_name') {
  …
}
```

---

## [SchemaCheckTrait::isViolationForIgnoredPropertyPath expects a ConstraintViolationInterface](https://www.drupal.org/node/3549625)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `$v` parameter for `Drupal\Core\Config\Schema\SchemaCheckTrait::isViolationForIgnoredPropertyPath` is now typehinted to the interface `Symfony\Component\Validator\ConstraintViolationInterface` rather than the `ConstraintViolation` concrete class.


Before
`Drupal\Core\Config\Schema\SchemaCheckTrait::isViolationForIgnoredPropertyPath(ConstraintViolation $v)`


After
`Drupal\Core\Config\Schema\SchemaCheckTrait::isViolationForIgnoredPropertyPath(ConstraintViolationInterface $v)`


Calling this method is not affected, but overriding this method in subclasses will need updating to match the trait.

---

## [#[RunTestsInSeparateProcesses] attribute is required for all Kernel, Functional and FunctionalJavascript tests](https://www.drupal.org/node/3548485)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

All Kernel, Functional and FunctionalJavascript tests must be executed in process isolation.


This has always been the case; up until PHPUnit 11 it was possible to require process isolation in the abstract base classes overriding the constructor, but since PHPUnit 12 this is no longer possible, so now


each concrete test class, Kernel, Functional and FunctionalJavascript, must specifiy a `#[RunTestsInSeparateProcesses]` attribute on the class level. 


Before:



```php
<?php

declare(strict_types=1);

namespace Drupal\Tests\announcements_feed\Functional;

use Drupal\dynamic_page_cache\EventSubscriber\DynamicPageCacheSubscriber;
use Drupal\Tests\BrowserTestBase;
use PHPUnit\Framework\Attributes\Group;

/**
 * Defines a class for testing pages are still cacheable with dynamic page cache.
 */
#[Group('announcements_feed')]
final class AnnouncementsCacheTest extends BrowserTestBase {
```

After:



```php
<?php

declare(strict_types=1);

namespace Drupal\Tests\announcements_feed\Functional;

use Drupal\dynamic_page_cache\EventSubscriber\DynamicPageCacheSubscriber;
use Drupal\Tests\BrowserTestBase;
use PHPUnit\Framework\Attributes\Group;
use PHPUnit\Framework\Attributes\RunTestsInSeparateProcesses;

/**
 * Defines a class for testing pages are still cacheable with dynamic page cache.
 */
#[Group('announcements_feed')]
#[RunTestsInSeparateProcesses]
final class AnnouncementsCacheTest extends BrowserTestBase {
```

---

## [The comment.new_comments_node_links route and CommentController::renderNewCommentsNodeLinks are deprecated](https://www.drupal.org/node/3543039)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `comment.new_comments_node_links` route and `Drupal\comment\Controller\CommentController::renderNewCommentsNodeLinks` controller are deprecated.


Use `history.new_comments_node_links` and `Drupal\history\Controller\HistoryController::renderNewCommentsNodeLinks` instead.

---

## [The block_content_add_list theme template is deprecated](https://www.drupal.org/node/3530643)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `block_content_add_list` theme hook and template are deprecated. You can use `entity_add_list` instead.


Before:



```php
// Where $types is a list of BlockContentType entities.
$build = ['#theme' => 'block_content_add_list', '#content' => $types];
```

After:



```php
// Where $bundles is an associative array keyed by the bundle machine name, with an array of values with keys label, description, and add_link. 
$build = [
  '#theme' => 'entity_add_list',
  '#bundles' => $bundles,
];
```
See `\Drupal\Core\Entity\Controller\EntityController::addPage`
[https://git.drupalcode.org/project/drupal/-/blob/11.x/core/lib/Drupal/Co...](https://git.drupalcode.org/project/drupal/-/blob/11.x/core/lib/Drupal/Core/Entity/Controller/EntityController.php#L151)

---

## [EntityController::addPage now requires the $request parameter](https://www.drupal.org/node/3546628)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`\Drupal\Core\Entity\Controller\EntityController::addPage` now requires the $request parameter, any child classes that are extending this method and calling its parent are required to add the new parameter before Drupal 12.

---

## [Hooks are no longer event listeners](https://www.drupal.org/node/3550627)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The module hook system no longer utilizes the event dispatcher for gathering hook implementations and therefore no longer tags automatically registered hook services as event listeners.


This significantly improves performance by reducing the container size and memory required for requests.


There are no changes to how OOP hooks are defined and used (exception: dynamically registered hooks through a service provider, see below)


Information about hooks is now stored in the hook_data key value store, this is considered internal information and directly accessing that information is not supported and there is no backwards compatibility contract, it's structure may change again in the future.


There is a new ImplementationList class to assist with sorting, filtering and looping over hook implementations.


Information about preprocess hooks has also been moved to the key value store and is no longer injected as a parameter into \Drupal\Core\Theme\Registry, instead, the key value factory is injected.


Backwards compatibility breaks
If you decorated or directly created the ModuleHandler class then you will need to update your code to match the new constructor and methods.


If you [utilized a service provider to inject hooks dynamically](https://www.drupal.org/project/field_encrypt/issues/3486465#comment-15851339) you will need to update your code. Instead, you will now need to update the hook_list key in the hook_data key value collection. Per previous comment, this is discouraged and not recommended. A possible alternative may be to check a runtime condition within the hook or [make the hook depend on a module once that is supported.](https://www.drupal.org/node/3548805)

---

## [Passing ModuleHandler and EntityTypeManager to CommentLinkBuilder is deprecated](https://www.drupal.org/node/3544527)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`\Drupal\comment\CommentLinkBuilder::__construct()` has been changed.  Passing the `module_handler` and `entity_type.manager` services as parameters is now deprecated.


Before:



```php
new CommentLinkBuilder(
  \Drupal::service('current_user'),
  \Drupal::service('comment.manager'),
  \Drupal::service('module_handler'),
  \Drupal::service('string_translation'),
  \Drupal::service('entity_type.manager'),
);
```

After:



```php
new CommentLinkBuilder(
  \Drupal::service('current_user'),
  \Drupal::service('comment.manager'),
  \Drupal::service('string_translation'),
);
```

---

## [Legacy hook functions are now attributed to the current module instead of the most specific match](https://www.drupal.org/node/3548085)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

When scanning for legacy procedural hooks Drupal will now by default attribute hook functions to the current module and only check for other modules if the function does not start with the current module name.


In the vast majority of cases this will have no effect since this is by far the most common way hooks are implemented.


Previously, it attributed hook functions to the longest module name that it could match them against, this resulted in hooks incorrectly being attributed to submodules if a function name overlapped with that name.


E.g. if both paragraphs and paragraphs_library were installed and the former implemented pararaphs_library_info_alter.
Instead of the paragraphs module and library_info_alter hook Drupal would register it as the paragraphs_library module and info_alter hook.


One side effect of this change is that a parent module can no longer implement a hook on behalf of a submodule if the parent module name prefixes the submodule since the first check will always match.


Note: Implementing hooks for another module is discouraged and may be removed in the future. It is now possible for a module to implement a hook more than once with OOP hooks and [#3524377: Allow to skip OOP hooks and services for modules that are not installed](https://www.drupal.org/project/drupal/issues/3524377) will allow to add a formal dependency on another module and register it conditionally.

---

## [Block content module no longer ships with field.storage.block_content.body.yml](https://www.drupal.org/node/3548819)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The block content module will no longer ship with field.storage.block_content.body.yml. Modules, recipes, or profiles that rely on that file can use block_content_storage_body_field until Drupal 13. But it's advised to just ship with your own copy of field.storage.block_content.body.yml.



```php
langcode: en
status: true
dependencies:
  module:
    - block_content
    - text
id: block_content.body
field_name: body
entity_type: block_content
type: text_with_summary
settings: {  }
module: text
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: true
custom_storage: false
```

---

## ["RSS publishing" settings form, system.rss config and RSS viewmode are removed from core](https://www.drupal.org/node/3519410)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The "RSS publishing" settings form and RSS viewmode are removed from core. The form was broken and it was not possible to to select that view if you even changed it. The dependencies of that view were also not up to par.


`system.rss` config is also removed from core. This means there is no default fallback configurable for RSS views for node and comment. The upgrade path will update the config, but to fix this make sure you select an explicit view mode (not default) for the RSS items in views.

---

## [The path alias preload cache has been removed](https://www.drupal.org/node/3532412)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The path alias preload cache was an optimization added in Drupal 6 which kept a cache of system paths per page for 24 hours, in order to load any path aliases in a single query rather than one by one.


This cache was less effective in modern Drupal versions due to render caching improvements, which mean that once render caches are warmed, path aliases are not looked up anyway or only a small subset of them.


From Drupal 11.3 this cache has been removed entirely from the AliasManager class. Modules that directly integrating with the path alias cache can remove any integration once they require Drupal 11.3 or higher.


Path alias preloading has been re-implemented via an alternative mechanism which uses PHP Fibers - when path aliases are requested within a Fiber, they will be added to a list of path aliases to preload, and then when the fiber is resumed, any aliases requested up until that point from other Fibers will be loaded at once. This can dramatically reduce the number of path alias lookups on cold caches, which the previous approach was unable to improve.


Contrib and custom modules may want to ensure their blocks or other rendered elements are correctly using placeholders and #lazy_builders in order to take full advantage of this and other performance improvements. See for example [https://www.drupal.org/node/3512518](https://www.drupal.org/node/3512518)

---

## [ListingEmpty area plugin for block_content views is deprecated](https://www.drupal.org/node/3336219)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

`\Drupal\block_content\Plugin\views\area\ListingEmpty` is deprecated.


An upgrade path has been added to remove it from existing views.


It previously provided a link to Add a content block when the view returned no results.



Before



After

---

## [Theme hook definitions for views plugins automatically define initial preprocess callback](https://www.drupal.org/node/3548185)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Implicitly discovered template_preprocess_HOOK() callbacks are being deprecated and replaced with explicitly defined initial callbacks.


Various views plugins have the unique ability to define a theme key in their plugin attribute, the views module then automatically defines a theme hook definition for them. That code now makes an assumption that an initial preprocess callback is defined in a specifically named method and class. This is only added if that method exists. Alternatively, plugins with such a callback can also use `hook_theme_registry_alter()` to define an alternative initial preprocess callback.


The initial preprocess callback is built with this pattern:



```php
Drupal\module_name\Hook\ModuleNameThemeHooks::preprocessCamelizedThemeHookName()
```

For example, the module `views_example_display` defines a display plugin with `theme: "views_display_example"`, then the resulting initial preprocess callback looks like this:



```php
namespace Drupal\views_example_display\Hook;

class ViewsExampleDisplayThemeHooks {

  public function preprocessViewsDisplayExample(array $variables) {

  }
}
```

This follows the adopted common pattern to put initial preprocess callbacks in the theme hooks class.


Note: This class is automatically registered as a service if there is at least one hook method in that class. Alternatively, it can be explicitly registered:



```php
services:
  Drupal\views_example_display\Hook\ViewsExampleDisplayThemeHooks:
    autowire: true
```

---

## [Defining theme_file for views plugins with a theme key in their plugin definition is deprecated](https://www.drupal.org/node/3548325)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Views plugins were able to specify a theme_file key in their plugin annotation to indicate an include file that contained their preprocess functions. 


Since template preprocess callbacks are now explicitly registered in the [hook_theme definition](https://www.drupal.org/node/3504125) and [views theme hooks defined for plugins support a standard preprocess callback](https://www.drupal.org/node/3548185), this key is deprecated.


The theme_file key is already not available in plugin attributes and support for it is removed in Drupal 12.0. As a minimal conversion, the functions can be moved to the .module file, or converted to the recommended OOP format.

---

## [_responsive_image_build_source_attributes(), responsive_image_get_image_dimensions(), responsive_image_get_mime_type(), _responsive_image_image_style_url() replaced with ResponsiveImageBuilder](https://www.drupal.org/node/3548329)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The following functions are replaced with methods on the class/service `\Drupal\responsive_image\ResponsiveImageBuilder`




Before
After


`_responsive_image_build_source_attributes()`
`\Drupal::service(ResponsiveImageBuilder::class)->buildSourceAttributes()`


`responsive_image_get_image_dimensions()`
`\Drupal::service(ResponsiveImageBuilder::class)->getImageDimensions()`


`responsive_image_get_mime_type()`
`\Drupal::service(ResponsiveImageBuilder::class)->getMimeType()`


`_responsive_image_image_style_url()`
`\Drupal::service(ResponsiveImageBuilder::class)->getImageStyleUrl()`

---

## [The comment/drupal.comment-new-indicator and comment/drupal.node-new-comments-link libraries have been deprecated](https://www.drupal.org/node/3537055)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `comment/drupal.comment-new-indicator` and `comment/drupal.node-new-comments-link` libraries are deprecated. Use `history/drupal.comment-new-indicator` and `history/drupal.node-new-comments-link` instead.

---

## [Migrate source plugins for legacy upgrade are deprecated](https://www.drupal.org/node/3533564)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Since the entire Drupal Migrate module will be removed in Drupal 12.0, many things need to be deprecated in preparation. The following classes/traits are deprecated for later removal.



- `Drupal\migrate_drupal\Plugin\migrate\DrupalSqlBase`

- `Drupal\migrate_drupal\Plugin\migrate\EmptySource`

- `Drupal\migrate_drupal\Plugin\migrate\I18nQueryTrait`


If you have migration classes that require these, either copy the contents of these classes into your code base or plan on upgrading to Drupal 11.

---

## [Node module no longer ships with field.storage.node.body.yml](https://www.drupal.org/node/3540814)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Starting in 11.3.x the node module will no longer ships with `field.storage.node.body.yml`.  Modules, recipes, or profiles that rely on that file can use `node_storage_body_field` until Drupal 13.  But it's advised to just ship with your own copy of `field.storage.node.body.yml`.



```php
langcode: en
status: true
dependencies:
  module:
    - node
    - text
id: node.body
field_name: body
entity_type: node
type: text_with_summary
settings: {  }
module: text
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: true
custom_storage: false
```

---

## [Page Cache Middleware uses Service Closure to speed up serving cached pages](https://www.drupal.org/node/3538740)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The page cache middleware is responsible for storing and serving cached pages. When a page is served from the cache, it shortcuts all inner middlewares including the HTTP kernel.


In order to further speed up cached pages, the page cache middleware was supposed to prevent the construction of services further up the stack. This has been the responsibility of the `responder` attribute on the `http_middleware` service tag. This mechanism has been implemented using lazy services during the development of Drupal 8.0 but was rendered ineffective by an unrelated change inadvertedly before 8.0 was even released.


The optimization has now been restored. The page cache middleware now takes a service closure as its first argument. Services further up the stack and their dependencies are only constructed in case of a cache `MISS`. On a cache `HIT`, a response is served with considerable reduced overhead compared to previous versions.


In order to transition from the (dysfunctional) `responder` attribute, the following changes are necessary:



- 
Drop the `responder` attribute from the `http_middleware` service tag:

```php
diff --git a/core/modules/page_cache/page_cache.services.yml b/core/modules/page_cache/page_cache.services.yml
index ec0f90d24b1..2f1ec6a4af9 100644
--- a/core/modules/page_cache/page_cache.services.yml
+++ b/core/modules/page_cache/page_cache.services.yml
@@ -8,7 +8,7 @@ services:
     class: Drupal\page_cache\StackMiddleware\PageCache
     arguments: ['@cache.page', '@page_cache_request_policy', '@page_cache_response_policy']
     tags:
-      - { name: http_middleware, priority: 200, responder: true }
+      - { name: http_middleware, priority: 200 }

   cache.page:
     class: Drupal\Core\Cache\CacheBackendInterface
```


- 
Modify the constructor of the middleware to take a closure as its first argument:

```php
/**
   * Constructs a PageCache object.
   *
   * @param \Symfony\Component\HttpKernel\HttpKernelInterface|\Closure $http_kernel
   *   The decorated kernel.
   * @param \Drupal\Core\Cache\CacheBackendInterface $cache
   *   The cache bin.
   * @param \Drupal\Core\PageCache\RequestPolicyInterface $request_policy
   *   A policy rule determining the cacheability of a request.
   * @param \Drupal\Core\PageCache\ResponsePolicyInterface $response_policy
   *   A policy rule determining the cacheability of the response.
   */
  public function __construct(HttpKernelInterface|\Closure $http_kernel, CacheBackendInterface $cache, RequestPolicyInterface $request_policy, ResponsePolicyInterface $response_policy) {
    if ($http_kernel instanceof HttpKernelInterface) {
      @trigger_error('Calling ' . __METHOD__ . '() without a service closure $http_kernel argument is deprecated in drupal:11.3.0 and it will throw an error in drupal:12.0.0. See https://www.drupal.org/node/3538740', E_USER_DEPRECATED);
      $http_kernel = static fn() => $http_kernel;
    }
    $this->httpKernel = $http_kernel;
    $this->cache = $cache;
    $this->requestPolicy = $request_policy;
    $this->responsePolicy = $response_policy;
  }
```

---

## [The advanced section of views' edit form will no longer collapse](https://www.drupal.org/node/3515212)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The third section or column of views' edit forms is no longer labelled as "Advanced" and is no longer collapsible.  Accordingly, the configuration setting to control the default collapse state has been removed.  This change was made for several reasons to enhance User Experience:



- To reduce the number of clicks required to edit a view

- To increase the visibility of important settings

- On desktop the collapsibility saves no space

- The settings aren't necessarily "advanced."


Providers of default configuration such as distributions should update `views.settings.yml` to remove the `advanced_column` key.

---

## [Twig rendering now uses yield](https://www.drupal.org/node/3546663)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

In preparation for Twig 4, and to support async rendering, Drupal core's Twig configuration now sets the `use_yield` option.


This should be a transparent change both for Twig templates themselves as well as any custom Twig integrations, however it is possible that complex twig implementations relying on output buffering could be affected.


Objects implementing `MarkupInterface` are now cast to a string in `TwigEnvironment::escapeFilter()` instead of returned as objects, this change should be completely transparent, but was necessary to ensure that mutable objects are rendered prior to any later mutations in complex templates. See the related issue for more details.


For more information in the Twig documentation see [https://twig.symfony.com/doc/3.x/deprecated.html#nodes](https://twig.symfony.com/doc/3.x/deprecated.html#nodes)

---

## [hook_ranking() has been renamed to hook_node_search_ranking()](https://www.drupal.org/node/2690393)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`hook_ranking()` has been deprecated in Drupal 11.3


The hook has been renamed to `hook_node_search_ranking()`, and it works exactly the same. Until Drupal 12, both hooks will be supported. 


Before



```php
<?php
/**
 * Implements hook_ranking().
 */
#[Hook('ranking')]
public function ranking(): array {
  ...
}
?>
```

After



```php
<?php
/**
 * Implements hook_node_search_ranking().
 */
#[Hook('node_search_ranking')]
public function ranking(): array {
  // Same function body as before; just change the function name.
}
?>
```

---

## [FileSystemInterface::basename() deprecated](https://www.drupal.org/node/3530869)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`FileSystemInterface::basename()` has been deprecated and will be removed in Drupal 13.


Use the PHP native `basename()` function as a replacement.



```php
-      \Drupal::service('file_system')->basename($path);
+      basename($path);
```

---

## [ExtensionMimeTypeGuesser no longer expects a $fileSystem argument](https://www.drupal.org/node/3534083)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `Drupal\Core\File\MimeType\ExtensionMimeTypeGuesser` no longer expects a $fileSystem argument as of Drupal 11.3.0 and the argument will be removed in Drupal 12.0.0.

---

## [Correctly display form description before the field prefix](https://www.drupal.org/node/3523777)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A form element's `#field_prefix` is intended to be displayed before and inline with the element in order to enable use cases such as displaying a currency symbol immediately before a number field.  When the ability to display element descriptions before the element was added, it was mistakenly placed between the prefix and input. 


This bug is now resolved for Core templates.  If you have overridden the `form-element.html.twig` template, then you should update your template accordingly.


Before:



```php
...
  {% if prefix is not empty %}
    <span class="field-prefix">{{ prefix }}</span>
  {% endif %}
  {% if description_display == 'before' and description.content %}
    <div{{ description.attributes.addClass(description_classes) }}>
      {{ description.content }}
    </div>
  {% endif %}
...
```
After:



```php
...
  {% if description_display == 'before' and description.content %}
    <div{{ description.attributes.addClass(description_classes) }}>
      {{ description.content }}
    </div>
  {% endif %}
  {% if prefix is not empty %}
    <span class="field-prefix">{{ prefix }}</span>
  {% endif %}
...
```

---

## [Using specific PDO drivers instead of PDOConnection on PHP 8.4+](https://www.drupal.org/node/3547277)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

To ensure compatibility with PHP 8.4 and PHP 8.5, this change updates the database connection classes to use the new driver-specific PDO connection classes (`Pdo\Mysql`, `Pdo\Pgsql`, `Pdo\Sqlite`) that were introduced in [PHP 8.4](https://wiki.php.net/rfc/pdo_driver_specific_subclasses). This avoids deprecation warnings that would otherwise appear [starting in PHP 8.5](https://wiki.php.net/rfc/deprecations_php_8_5#deprecate_driver_specific_pdo_constants_and_methods).


The changes primarily involve conditionally using the new PDO driver classes when the PHP version is 8.4 or higher. This affects the `mysql`, `pgsql`, and `sqlite` database drivers. For the SQLite driver, the code has also been updated to use the new [createFunction()](https://www.php.net/manual/ru/pdo-sqlite.createfunction.php) syntax with first-class callable syntax when running on PHP 8.4+

---

## [Utility method to replace unserialize() in SqlContentEntityStorage()](https://www.drupal.org/node/3495417)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The method `handleNullableFieldUnserialize()` is added to `SqlContentEntityStorage` and replaces the use of `unserialize()` throughout the class.  Unlike `unserialize()`, this method returns NULL if the input value is NULL..


This affects fields using `\Drupal\Core\Entity\Sql\SqlContentEntityStorage` that have serialized columns that can be NULL if they are not required. 


Before
`\Drupal\Core\Entity\Sql\SqlContentEntityStorage` automatically unserializes these columns, which causes an E_WARNING and a deprecation warning in the event of a NULL value. For example,



```php
Deprecated function: unserialize(): Passing null to parameter #1 ($data) of type string is deprecated in Drupal\Core\Entity\Sql\SqlContentEntityStorage->loadFromSharedTables()
```
After
The method `handleNullableFieldUnserialize()` returns NULL if the value is NULL. If not NULL the value is passed to `unserialize()`. No warning is issued.

---

## [Inline blocks are no longer editable via the block UI](https://www.drupal.org/node/3527795)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Inline blocks can no longer be edited or deleted via the normal block content UI routes at `/admin/content/block/ID` and `/admin/content/block/ID/delete`.


Managing block content entities via this method can break the block in context of the Layout it is attached to. Inline blocks should only ever be managed via the Layout page of the entity it is attached to.

---

## [Wrapper format to use HtmxRenderer added](https://www.drupal.org/node/3544666)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Drupal uses wrapper format query parameters to adjust the format of responses on a given path. Drupal 11.3 adds a wrapper format for use with HTMX.  This format returns the main content and needed JS/CSS assets in a very simple document.


The template used for the markup is very streamlined:



```php
<html>
  <head>
    <title>{{ title }}</title>
    <css-placeholder token="{{ placeholder_token }}">
    <js-placeholder token="{{ placeholder_token }}">
    <js-bottom-placeholder token="{{ placeholder_token }}">
  </head>
  <body>{{ content }}</body>
</html>
```

To use this format, add the query parameter to your url:



```php
$options = [
      'query' => [
        MainContentViewSubscriber::WRAPPER_FORMAT => 'drupal_htmx',
      ],
    ];
$url = Url::fromRoute('config.export_single', [], $options);
(new Htmx())
  ->post($url)
  ->select('*:has(>select[name="config_name"])')
  ->target('*:has(>select[name="config_name"])')
  ->swap('outerHTML')
  ->applyTo($form['config_type']);
```

---

## [Ajax subsystem now includes HTMX](https://www.drupal.org/node/3539472)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

HTMX is designed as an extension of HTML.  It is therefore a declarative markup system that uses attributes.



```php
<button hx-get="/contacts/1" hx-target="#contact-ui">
    Fetch Contact
</button>
```

Instead, we have declarative attributes much like the
`href` attribute on anchor tags and the `action`
attribute on form tags. The `hx-get` attribute tells htmx:
“When the user clicks this button, issue a `GET` request to
`/contacts/1`.” The `hx-target` attribute tells
htmx: “When the response returns, take the resulting HTML and place it
into the element with the id `contact-ui`.”



from [Hypermedia Systems](https://hypermedia.systems/hypermedia-a-reintroduction/#_hypermedia_driven_applications)


HTMX is just as happy with `data-hx-get` and so we will maintain standard markup in our implementation.


HTMX uses over [30 such attributes](https://htmx.org/reference/). The other control surface for HTMX is a set of response headers. HTMX supports [11 custom response headers](https://htmx.org/reference/#response_headers).


New Classes


`\Drupal\Core\Htmx\Htmx` provides methods to build the 30 custom attributes used by HTMX.  This class also provides methods to implement the headers that HTMX declares as part of its controls. Each method is documented with a short explanation and a link to the attribute or header documentation.


Another way of understanding this class is to regard the attributes and headers of HTMX and their expected data as an interface. `\Drupal\Core\Htmx\Htmx` implements this interface within Drupal and modifies our normal render array to produce the needed attributes and headers.  This class intentionally does not have a PHP interface given this consideration of the native HTMX controls as the interface for this class. This means that the class is [internal by policy](https://www.drupal.org/about/core/policies/core-change-policies/bc-policy#public-methods) and open to change but it will only change if the HTMX library changes.


Developers familiar with the `\Drupal\Core\Cache\CacheableMetadata` class and its interaction with render arrays cache key-value pairs will find the same pattern implemented in `Htmx`.


`\Drupal\Core\Render\Hypermedia\HtmxLocationResponseData` is added as an optional data object to manage the structured data that can be used with the HTMX location header.


Revised Class


`\Drupal\Core\Form\FormBuilder` has been revised to be aware of HTMX requests in parity with its awareness of Ajax API requests.  HTMX attributes have been added to the form_build_id element so that this value is dynamically updated when responding to an HTMX request.  This requires that the custom headers added by HTMX be available on the request object.  Hosting environments that filter non-standard headers should be adjusted to permit the request headers used by HTMX which are listed below.


Required Request Headers


HTMX adds the [following headers](https://htmx.org/reference/#request_headers).  Proxies or hosting setups that filter out non-standard headers should be adjusted to permit these headers:



- HX-Boosted

- HX-Current-URL

- HX-History-Restore-Request

- HX-Prompt

- HX-Request

- HX-Target

- HX-Trigger-Name

- HX-Trigger


References
[Htmx documentation](https://htmx.org/reference/)
[Little HTMX Book](https://littlehtmxbook.com/) is a brief exploration of using HTMX.
[Hypermedia Systems](https://hypermedia.systems/book/contents/) is an extended exploration of htmx as an implementation of hypermedia.


Before (Ajax API)
Ajax API and its predecessors has been our tool for adding this type of interactivity for about 15 years.



```php
$config_types = [
   'system.simple' => $this->t('Simple configuration'),
] + $entity_types;
 $form['config_type'] = [
  '#title' => $this->t('Configuration type'),
  '#type' => 'select',
  '#options' => $config_types,
  '#default_value' => $config_type,
  '#ajax' => [
    'callback' => '::updateConfigurationType',
    'wrapper' => 'js-config-form-wrapper',
  ],
];
```

Before
HTMX was added as a dependency in 11.2 so it has been possible to directly add htmx specific attributes to a render array. For example:



```php
use Drupal\Core\Url;

/*
 * - Send a POST request to the form URL.
 * - Select the wrapper element of the config_name <select> element from the response.
 * - Target the wrapper element of the config_name <select> in the rendered form for replacement.
 * - Use the outerHTML strategy, which is to replace the whole tag.
 */
);
$form['config_type'] = [
  '#title' => $this->t('Configuration type'),
  '#type' => 'select',
  '#options' => $config_types,
  '#default_value' => $config_type,
  '#attributes' => [
    'data-hx-post' => '/admin/config/development/configuration/single/export',
    'data-hx-select' => '*:has(>select[name="config_name"])',
    'data-hx-target' => '*:has(>select[name="config_name"])',
    'data-hx-swap' => 'outerHTML',
  ],
  '#attached' => [
    'library' => ['core/drupal.htmx'],
  ]
];

// Update the url when the config name selector dynamically changes.
if (!empty($default_type) && !empty($default_name)) {
  $push = Url::fromRoute('config.export_single', [
    'config_type' => $default_type,
    'config_name' => $default_name,
  ]);
  $form['config_name']['#attached']['http_header'][] = [
    'HX-Push-Url',
    $push->toString(),
    TRUE,
  ];
}
```

After



```php
use Drupal\Core\Htmx\Htmx;
use Drupal\Core\Url;

$form['config_type'] = [
  '#title' => $this->t('Configuration type'),
  '#type' => 'select',
  '#options' => $config_types,
  '#default_value' => $config_type,
];

$form_url = Url::fromRoute(
  'config.export_single',
  ['config_type' => $config_type, 'config_name' => $config_name]
);
(new Htmx())->post($form_url)
  ->select('*:has(>select[name="config_name"])')
  ->target('*:has(>select[name="config_name"])')
  ->swap('outerHTML')
  ->applyTo($form['config_type']);

// Update the url when the config name selector dynamically changes.
if (!empty($default_type) && !empty($default_name)) {
  $push = Url::fromRoute('config.export_single', [
    'config_type' => $default_type,
    'config_name' => $default_name,
  ]);
(new Htmx())
  ->pushUrlHeader($push)
  ->applyTo($form['config_name']);
}
```

Whenever a method calls for a `Url` object, the cacheable metadata emitted by rendering the object to string is also collected and merged to the render array by the `::applyTo` method.


A static method `Htmx::createFromRenderArray` is provided which takes a render array as input and builds an new instance of `Htmx` with all the HTMX specific attributes and headers loaded from the array.

---

## [Drupal Scaffold composer plugin generates a new \Drupal\DrupalInstalled class](https://www.drupal.org/node/3531162)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The container cache key has been updated to use a hash of the all the versions of the packages installed by composer, including Drupal contrib, non-Drupal packages, and the root project package. The hash is written to the `\Drupal\DrupalInstalled` class during composer autoload dumping. The `DrupalInstalled.php` file is located in the vendor/drupal directory. The `\Drupal\DrupalInstalled::VERSIONS_HASH` constant can be used to construct any cache key that needs to change when code changes.


This also means that contrib and custom modules do not need to add empty update or post update hooks to force a container rebuild to load new services. 


The drupal/core-composer-scaffold plugin is now a dependency of drupal/core. If you have disabled the plugin then you must add the following pre-autoload-dump script to your root composer.json



```php
"scripts": {
        "pre-autoload-dump": "Drupal\\Composer\\Plugin\\Scaffold\\Plugin::preAutoloadDump"
    },
```

---

## [node_type_get_names is deprecated](https://www.drupal.org/node/3534849)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`node_type_get_names` is deprecated, a new method on the `entity_type.bundle.info` service has been added to provide the same functionality for any entity type.


Before:
`$type_names = node_type_get_names();`


After:
`$type_names = \Drupal::service('entity_type.bundle.info')->getBundleLabels('node');`

---

## [Drupal\node\NodeStorage::revisionIds, ::userRevisionIds, and ::countDefaultLanguageRevisions are deprecated](https://www.drupal.org/node/3519187)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

All methods on `Drupal\node\NodeStorage` except `::clearRevisionsLanguage` are now deprecated, some functions have been replaced by entity queries and some have no replacement:


NodeStorage::revisionIds
Before:



```php
$revision_ids = $node_storage->revisionIds($node);
```

After:



```php
$query = \Drupal::entityQuery('node')->allRevisions()->condition('nid', $node->id())->accessCheck(FALSE);
$revision_ids = array_keys($query->execute());
```
NodeStorage::userRevisionIds
Before:



```php
$revision_ids = $node_storage->userRevisionIds($account);
```

After:



```php
$query = \Drupal::entityQuery('node')->allRevisions()->accessCheck(FALSE)->condition('uid', $account->id());
$revision_ids = array_keys($query->execute());
```
NodeStorage::countDefaultLanguageRevisions
Has been removed with no replacement. This was unused in Drupal core.

---

## [UserSession name property visibility changed to protected](https://www.drupal.org/node/3513877)

- **Version**: 11.3.0
- **Branch**: 11.3,x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `UserSession::name` property visibility has been changed to protected.


Custom and contrib code will need to be updated to use the `getAccountName()` method instead.


Before:



```php
$userName = $user->name;
```

After:



```php
$userName = $user->getAccountName();
```

---

## [DRUPAL_DISABLED, DRUPAL_OPTIONAL, DRUPAL_REQUIRED are deprecated](https://www.drupal.org/node/3448089)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The constants `DRUPAL_DISABLED, DRUPAL_OPTIONAL, DRUPAL_REQUIRED` have been deprecated.


Use of these constants have been replaced with `enum` classes in the respective modules.



- `comment`

- `link`

- `node`


The new, equivalent enums have kept the same integer values.


See:



- `core/modules/comment/src/CommentPreview.php`

- `core/modules/link/src/LinkTitleVisibility.php`

- `core/modules/node/src/NodePreview.php`


Either use the appropriate new enum or create a custom enum.

---

## [Entity operations methods can now add cacheable metadata](https://www.drupal.org/node/3533080)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Several entity operation based methods have added a new `\Drupal\Core\Cache\CacheableMetadata $cacheability` parameter:


`\Drupal\Core\Entity\EntityListBuilder::getOperations()`
`\Drupal\Core\Entity\EntityListBuilder::getDefaultOperations()`
`hook_entity_operation`
`hook_entity_operation_alter`


This is especially useful for adding cacheable metadata from access checks for operations. EntityListBuilder now does this by default for the edit and delete operations.


For hooks, the new parameter can be added to new and existing implementations:



```php
/**
 * Implements hook_entity_operation().
 */
#[Hook('entity_operation')]
public function entityOperation(\Drupal\Core\Entity\EntityInterface $entity, \Drupal\Core\Cache\CacheableMetadata $cacheability): array {
  $access = $entity->access('foo', NULL, TRUE);
  $cacheability->addCacheableDependency($access);
  if ($access->isAllowed()) {
    $operations['my_operation'] = [
      'title' => 'My operation',
      'weight' => 20,
      'url' => $entity->toUrl('bar'),
    ];
  }
}
```

Views and EntityListBuilder also automatically applies this cacheable metadata to the render output of its Operations field.

---

## [PathBasedBreadcrumbBuilder no longer renders an empty breadcrumb for paths without a title](https://www.drupal.org/node/3541758)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Previously, the PathBasedBreadcrumbBuilder would add breadcrumbs for paths in the trail that contained an empty title, resulting in a breadcrumb with no link. Now these paths are excluded from the list of breadcrumbs entirely.

---

## [Views based theme suggestions for node and comment templates deprecated](https://www.drupal.org/node/3541462)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Views used to provide theme suggestions to theme nodes or comments differently based on the view that is displaying them.


There are known cache bugs, as those theme suggestions do not come with approriate cacheability metadata, which means that they will not work reliably.


These theme suggestions are now deprecated. Use different view modes instead when different views should display nodes differently.

---

## [The view variable passed to comment templates is deprecated](https://www.drupal.org/node/3541463)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

The `view` variable is not safe to use in comment templates as no cacheability metadata is provided for it. The template may be built for a certain view and a cached response is returned for a different view. It must not be relied upon. Hence, the `view` variable passed to the comment templates as `$variables['view']` is deprecated in 11.3.0 and will be removed in 13.0.0. 


This change aims to streamline the theming process and ensure consistency across node templates.

---

## [Drupal\comment\Plugin\views\field\NodeNewComments is deprecated](https://www.drupal.org/node/3542850)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`\Drupal\comment\Plugin\views\field\NodeNewComments` is deprecated, use `\Drupal\history\Plugin\views\field\NodeNewComments` instead. The plugin ID has not changed, so this should not affect any configuration using the plugin.

---

## [taxonomy_term_is_page and the page taxonomy-term.html.twig variable are deprecated](https://www.drupal.org/node/3542527)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The 'page' variable for taxonomy-term.html.twig is deprecated, the 'full' view mode can be checked instead.


Additionally, the taxonomy_term_is_page() procedural function is also deprecated.

---

## [node_is_page and the page node.html.twig variable are deprecated](https://www.drupal.org/node/3458593)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The 'page' variable for node.html.twig is deprecated, the 'full' view mode can be checked instead.


This results in a consistent behavior of the full view mode, which will no longer display the node title. This may result in some unexpected behavior if sites used views, formatters or custom code that relied on this behavior. It also fixes several bugs with missing or duplicated page titles, for example on the revision page.


Additionally, the node_is_page() procedural function is also deprecated.

---

## [Use the getToolkitId() method instead of the toolkitId property when in an ImageFactory subclass](https://www.drupal.org/node/3541926)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The toolkitId property in ImageFactory is no longer initialized in the constructor but instead only when accessed.


Subclasses should call getToolkitId():


Before:



```php
$this->toolkitId
```

After:



```php
$this->getToolkitId()
```

The getToolkitId() method has existed since Drupal 8.0.

---

## [node_mass_update() is deprecated](https://www.drupal.org/node/3533315)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`node_mass_update()` is deprecated.


Use `\Drupal::service(\Drupal\node\NodeBulkUpdate::class)->process()` instead.


Example



```php
\Drupal::service(NodeBulkUpdate::class)->process($nodes, ['status' => 0], NULL, TRUE);
```

`core/modules/node/node.admin.inc` has been deprecated as well, since `node_mass_update()` is the only function in that file.

---

## [CommentTestBase::setCommentPreview() now takes a CommentPreviewMode enum instead of an int for $mode](https://www.drupal.org/node/3538678)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Instead of passing an integer for the `$mode` parameter to the `CommentTestBase::setCommentPreview()` method, use the new `Drupal\comment\CommentPreviewMode` enum.


Before
 `$this->setCommentPreview(1);`


After
`$this->setCommentPreview(Drupal\comment\CommentPreviewMode::Optional);`

---

## [Recipe input config and env source elements can now have fallbacks set](https://www.drupal.org/node/3540260)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

When a recipe asks for input, it must contain a `source` element, which can be one of `config`, `env`, or `value`.


The `config` and `env source` elements can contain fallbacks which is helpful for when a config file is not available yet, or an environment variable does not exist.


config source example:



```php
input_name:
  default:
    source: config
    config: ['foo.settings', 'baz']
    fallback: bar
```

environment source example:



```php
input_name:
  default:
    source: env
    env: 'RUNTIME_ENVIRONMENT'
    fallback: 'DEV'
```

fallback allows for null:



```php
input_name:
  default:
    source: env
    env: 'FOO'
    fallback: null
```

`value` sources remain required and can not have a fallback.

---

## [Added support for `@>` as a shorthand for `!service_closure` in services.yml files](https://www.drupal.org/node/3527390)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

You can now use `@>` as a shorthand for `!service_closure` in services.yml files.


Original (still supported) syntax:



```php
example_service_closure:
    class: \Drupal\Core\ExampleClass
    arguments: [!service_closure '@example_service_1']
```
New shorthand syntax:



```php
example_service_closure_shorthand:
    class: \Drupal\Core\ExampleClass
    arguments: ['@>example_service_1']
```

---

## [MySQL's findTables() will no longer find database views](https://www.drupal.org/node/3531733)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The MySQL driver's schema object has been fixed to behave the same as SQLite and PostgresQL's and no longer list database views in the output of `\Drupal::database()->schema()->findTables()`. Additionally, `\Drupal::database()->schema()->tableExists('database view')` will now return FALSE.

---

## [getPreviewMode() and setPreviewMode() on NodeTypeInterface now expect a NodePreviewMode enum](https://www.drupal.org/node/3538666)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`NodeTypeInterface::getPreviewMode` and `NodeTypeInterface::setPreviewMode` now interact with the `\Drupal\node\NodePreviewMode` enum instead of integer values.


A new deprecated parameter `$returnAsInt` has been added to `getPreviewMode` to maintain backwards compatibility. Passing FALSE to this will return the NodePreviewMode enum instead.


Passing an integer to `setPreviewMode` is now deprecated, use the NodePreviewMode enum instead.

---

## [Theme suggestions can now be deprecated](https://www.drupal.org/node/3535678)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

It is now possible to deprecate theme suggestions. Using the special __DEPRECATED key, list each deprecated theme suggestion, next to its definition with the deprecation message as value.



```php
/**
 * Implements hook_theme_suggestions_alter().
 */
function MODULE_suggestions_alter(array &$suggestions, array $variables, $hook): void {
  if ($hook == 'theme_test_suggestion_provided') {
    // Add a deprecated suggestion.
    $suggestions[] = 'theme_test_suggestion_provided__deprecated';
    $suggestions['__DEPRECATED']['theme_test_suggestion_provided__deprecated'] = 'Theme suggestion theme_test_suggestion_provided__deprecated is deprecated in drupal:X.0.0 and is removed from drupal:Y.0.0. This is a test.';
  }
}
```

---

## [node_access_view_all_nodes is deprecated](https://www.drupal.org/node/3038909)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`node_access_view_all_nodes` is deprecated. 


its functionality was combined into `\Drupal\node\NodeGrantDatabaseStorage::checkAll()`.


Before

```php
$view_all_nodes = node_access_view_all_nodes();

// Reset the internal cache.
drupal_static_reset('node_access_view_all_nodes');
```
After

```php
$view_all_nodes = \Drupal::entityTypeManager()->getAccessControlHandler('node')->checkAllGrants($account ?? \Drupal::currentUser())

// Reset the internal cache.
\Drupal::service('node.view_all_nodes_memory_cache')->deleteAll();
```

Calling `drupal_static_reset` with `node_access_view_all_nodes` is also deprecated.


2 constructors also have new arguments:


`NodeAccessGrantsCacheContext::__construct()` now requires an `$entityTypeManager`
`NodeGrantDatabaseStorage::__construct()` now requires a `$memoryCache` which is the new `'node.view_all_nodes_memory_cache'` service.

---

## [The Ban module is deprecated](https://www.drupal.org/node/3533895)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The Ban module is deprecated in Drupal 11.3.0 and will be removed in Drupal 12.0.0. 


If you want to keep using the functionality provided by the Ban module, read the [recommendations for Ban](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-ban).

---

## [Entity types used in kernel tests need all dependent modules to be installed in order to be discovered](https://www.drupal.org/node/3539877)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

For any entity type used in a kernel test, all modules that have code that entity type class inherits from must be explicitly installed in the kernel test in order for the entity type to be discovered.


Using `Node` as an example, any kernel test that uses nodes must have both the `node` and `user` modules installed.


Before



```php
class NodeExampleTest extends KernelTestBase {

  /**
   * {@inheritdoc}
   */
  protected static $modules = ['node'];
  ...

  public function testExample(): void {
    $node = \Drupal::entitTypeManager()->loadStorage('node')->create([
      ...
    ]);
    ...
  }
  
}
```

After



```php
class NodeExampleTest extends KernelTestBase {

 /**
   * {@inheritdoc}
   */
  protected static $modules = ['node', 'user'];
  ...

  public function testExample(): void {
    $node = \Drupal::entitTypeManager()->loadStorage('node')->create([
      ...
    ]);
    ...
  }
  
}
```

Adding `user` to `$modules` in this example is necessary because the `Node` entity type class uses `Drupal\user\EntityOwnerTrait` in the `user` module. And while the `node` module has `drupal:text` defined as a dependency in `node.info.yml`, and thus `drupal:filter` and `drupal:user` as part of the dependency chain, in kernel tests, modules listed in the `$modules` property are installed without implictly installing their dependencies.

---

## [getDependencies() and setDependencies() methods have been added to Drupal\Component\Plugin\Attribute\AttributeInterface](https://www.drupal.org/node/3523753)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Two new methods `getDependencies()` and `setDependencies()` have been added to `Drupal\Component\Plugin\Attribute\AttributeInterface`. 


Plugin discovery by PHP attributes now uses Reflection to determine what dependencies the plugin class has, based on the class hierarchy. These dependencies include the fully qualified class names of all the ancestor classes the class extends, all the interfaces the class implements, and all the traits used by the class and its ancestor classes. From there, providers (in other words, modules) for the classes, interfaces, and traits are derived.


These dependencies are passed into the attribute object during discovery so that they can be made available in plugin definitions. `getDependencies()` and `setDependencies()` work with arrays that are indexed by the type of dependency (`class`, `interface`, `trait`, or `provider`). The values for each type are the fully qualified names, except for `provider` dependencies, which are the just the providers' machine names. Though generally in plugin processing, only the provider dependencies are passed into the attribute object.


For a plugin whose definition is an array `$definition`, the provider dependencies are available via the `$definition['dependencies']['provider']` if any dependencies are set. Plugin types that use object definitions will need ways to access the dependencies to be implemented as needed.

---

## [block_content_add_body_field is deprecated](https://www.drupal.org/node/3535528)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `block_content_add_body_field` method is now deprecated, there is no direct replacement.


If you need to create block content types in tests with a body field, use `Drupal\Tests\block_content\Traits\BlockContentCreationTrait::createBlockContentType` which will automatically add a body field or use `Drupal\Tests\field\Traits\BodyFieldCreationTrait`

---

## [Magically named cancel functions in Views UI forms have been deprecated](https://www.drupal.org/node/3536715)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Previously forms that call `\Drupal\views_ui\ViewsUI::getStandardButtons()` could have a custom cancel submit handler automatically applied if it was named `$form_id . '_cancel'`. If this function did not exist the cancel button would use `ViewUI::standardCancel()` instead.


These magically named cancel functions have been deprecated. Specify a submit handler in a class method instead.


Before:



```php
<?php

function my_form_id_cancel($form, FormStateInterface $form_state) {
  // Custom handler.
}

class MyForm extends ViewsFormBase {

  public function buildForm(array $form, FormStateInterface $form_state) {
    $view->getStandardButtons($form, $form_state, 'my_form_id');
  }

}
?>
```

After:



```php
<?php
class MyForm extends ViewsFormBase {

  public function buildForm(array $form, FormStateInterface $form_state) {
    $view->getStandardButtons($form, $form_state, 'my_form_id');
    $form['actions']['cancel']['#submit'] = [[$this, 'cancelSubmit']],
  }

  public function cancelSubmit($form, FormStateInterface $form_state) {
    // Custom handler.
  }

}
?>
```

---

## [Promoted/Sticky fields are hidden by default for new Node Types](https://www.drupal.org/node/3518643)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

For new content (node) types created after 11.3.0, the Promoted to homepage and Sticky fields will be hidden by default on the content type's form display. Existing node types will be unaffected.


Site builders can show these fields by going to the Manage form display page for the node type and dragging them out of the Disabled section.

---

## [comment_uri() is deprecated](https://www.drupal.org/node/3384294)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The function `comment_uri()` is deprecated use `\Drupal\comment\Entity\Comment::permalink()` or `Comment::toUrl()` with `fragment` option instead.

---

## [Added --phpunit-configuration argument to run-tests.sh](https://www.drupal.org/node/3536709)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new `--phpunit-configuration <path>` argument is added to the test runner script `run-tests.sh`


The `<path>` value is passed unabridged to PHPUnit's CLI `--configuration` argument when run-tests is spawning subprocesses involving running a PHPUnit command.


From [PHPUnit documentation](https://docs.phpunit.de/en/11.5/textui.html):



`-c|--configuration <file>`


Configure PHPUnit’s test runner using an XML configuration file. This is not required when the configuration file that is to be used is located in the current working directory and is named `phpunit.xml`, `phpunit.dist.xml`, or `phpunit.xml.dist`.

---

## [Publishing a workspace will update the changed time for its entities](https://www.drupal.org/node/3531039)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Starting with Drupal 11.3.0, publishing a workspace will update the changed timestamp for its entities.


Calling the `\Drupal\workspaces\WorkspacePublisher` or `\Drupal\workspaces\WorkspaceOperationFactory` constructor without the `$time` argument is deprecated. The `$time` argument is required in Drupal 12.

---

## [ModuleHandler::loadAllIncludes() is deprecated](https://www.drupal.org/node/3536432)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`ModuleHandler::loadAllIncludes()` is deprecated, there is no direct replacement.


As a replacement, `ModuleHandler::loadInclude()` can be called in a loop for modules. 


Be aware that `ModuleHandler::loadInclude(`) and all include files will eventually be deprecated.

---

## [node_add_body_field() is deprecated](https://www.drupal.org/node/3516778)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The `node_add_body_field` method was only used in test code by Drupal core.


The function is now deprecated, there is no direct replacement.


If you need to create content types in tests with a body field, use `ContentTypeCreationTrait::createContentType` which will automatically add a body field.


If you need to add body fields for other entity types, there is also a new `BodyFieldCreationTrait` to add the same type of field to any bundle.


Otherwise you can create the field config directly:



```php
$field_storage = FieldStorageConfig::loadByName('node', 'body');
  $field = FieldConfig::loadByName('node', $bundle, 'body');
  if (!$field) {
    $field = FieldConfig::create([
      'field_storage' => $field_storage,
      'bundle' => $bundle,
      'label' => $label,
      'settings' => [
        'display_summary' => TRUE,
        'allowed_formats' => [],
      ],
    ]);
    $field->save();
  }
```

---

## [node_reindex_node_search() is deprecated](https://www.drupal.org/node/3533632)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The procedural function `node_reindex_node_search()` has been deprecated. Its logic has been moved to `\Drupal\node\NodeSearchHooks` implementation.


If your code directly called `node_reindex_node_search()`, replace it with a call to:



```php
\Drupal::service('search.index')->markForReindex('node_search', $nid);
```

---

## [file_system_settings_submit() is deprecated](https://www.drupal.org/node/3534099)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`file_system_settings_submit() `is deprecated and moved to `\Drupal\file\Hook\FileHooks::settingsSubmit()`

---

## [template_preprocess_node_add_list and template_preprocess_node are deprecated](https://www.drupal.org/node/3533060)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`template_preprocess_node_add_list` and `template_preprocess_node` are deprecated.


`template_preprocess_node_add_list` is replaced with `\Drupal::service(NodeThemeHooks::class)->preprocessNodeAddList($variables)`.


`template_preprocess_node` is replaced with `\Drupal::service(NodeThemeHooks::class)->preprocessNode($variables)`.


See [https://www.drupal.org/node/3504125](https://www.drupal.org/node/3504125) for more information.

---

## [Promoted to front page now defaults to FALSE for new content types](https://www.drupal.org/node/3517642)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Before this change, when creating a new content type, the default value for the "Promoted to front page" field on a Node was TRUE. This field is configurable per Node Type under the Publishing Options when editing the Node Type. 


Changing the setting on a Node Type creates or updates a base field override configuration entity for that node type's promote field to override the default.


Existing Node Types that were created prior to 11.3.0, and were using the default value of TRUE, will have base field override configuration created to maintain the previous default setting.


New content types created after 11.3.0 will default to FALSE.

---

## [`content:export` command added to help with recipe development](https://www.drupal.org/node/3533854)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Recipes have been able to include default content since they were first introduced to core. However, creating that default content has not been supported by core; it was necessary to use the contributed [Default Content](https://www.drupal.org/project/default_content) module to export content from Drupal into the format used by recipes.


As of [#3532694: Add a command-line utility to export content in YAML format](https://www.drupal.org/project/drupal/issues/3532694), Drupal core now includes a command-line tool to export content in that format. It can export a single entity at a time, but it is also possible to export the dependencies of the entity automatically (for example, images attached to it or taxonomy terms it references).


How to use
To use it, run the following from the Drupal root:



```php
php core/scripts/drupal content:export <ENTITY_TYPE_ID> <ENTITY_ID>
```

For example: `php core/scripts/drupal content:export taxonomy_term 40`


This will output a dump of the entity's data in YAML format. You can save it to a file like this: `php core/scripts/drupal content:export taxonomy_term 40 > recipes/my_recipe/content/taxonomy_term/some-tag.yml`


As of [#3532951: Support exporting content and its dependencies to a folder structure on disk](https://www.drupal.org/project/drupal/issues/3532951), you can also export the entity to a directory:



```php
php core/scripts/drupal content:export node 42 --dir=my-content
```
This will create `my-content/node/UUID_OF_NODE_42.yml`.


You can export the entity and all of its referenced entities, recursively, to a directory:



```php
php core/scripts/drupal content:export node 42 --with-dependencies --dir=content
```
This will export the content to a directory called `content`, divided by entity type into additional subdirectories. It's similar to the `drush dcer` command, from the contrib Default Content module.


It is possible to export all entities of a particular type, optionally filtered by bundle. For example, to export all `blog` content:



```php
php core/scripts/drupal content:export node --bundle=blog --with-dependencies --dir=content
```

Or multiple bundles, by passing the `--bundle` option more than once:



```php
php core/scripts/drupal content:export node --bundle=blog --bundle=faculty --with-dependencies --dir=content
```

You can leave out the `--bundle` option entirely to just export all entities of a particular type:



```php
php core/scripts/drupal content:export media --dir=content
```
How to integrate with the API
The export command supports core field types, but modules that provide their own field types may need to write a little code to integrate with this command.


You can attach a callback to a particular field type (or a particular field) by writing an event subscriber that listens to `\Drupal\Core\DefaultContent\PreExportEvent`, which is dispatched before an entity is exported. The subscriber should call the event's `setCallback()` method, which takes either a field name or a field type (such as `field_item:image` -- if using a field type, the `field_item:` prefix must be present), and a callback that takes two arguments: the field item being exported (`\Drupal\Core\FieldItemInterface`) and an object that collects information about the entity being exported (`\Drupal\Core\DefaultContent\ExportMetadata`). It should return the exported values for that field item, or NULL if the item shouldn't be exported.


For examples, see the following classes:



- `\Drupal\Core\DefaultContent\Exporter`

- `\Drupal\link\EventSubscriber\DefaultContentSubscriber`


The event can also be used to opt a specific field out of being exported:



```php
$event->setExportable('field_name', FALSE);
```

Computed fields are not exported by default, but you can use that same method to opt them in:



```php
$event->setExportable('some_computed_field', TRUE);
```
How to use the API
You can export an entity programmatically using the new `Exporter` service:



```php
use Drupal\Core\DefaultContent\Exporter;

$node = Node::load(32);
$exporter = \Drupal::service(Exporter::class);
$exported_content_as_yaml = (string) $exporter->export($node);

// Export to a file: creates /path/to/content/directory/node/UUID.yml.
$exporter->exportToFile($node, '/path/to/content/directory');

// Export to a directory with all dependencies, divided into subdirectories.
$exporter->exportWithDependencies($node, '/path/to/content');
```

---

## [node_get_type_label is deprecated](https://www.drupal.org/node/3533301)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`node_get_type_label` is deprecated.


If on 11.3 or above
Use `$node->getBundleEntity()->label()` instead.


If on 11.2 or below
Use `$node->get('type')->entity->label()` instead.

---

## [CKEditor 5 now offers a UI for setting list type](https://www.drupal.org/node/3529709)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

CKEditor 5 32.0.0 added a UI for setting a list type and now Drupal uses it:





Sites with text formats that currently allow the `type` attribute on `ul` or `ol` elements through either unrestricted formats (such as "Full HTML") or manually editable HTML tags will automatically allow the user to choose a list style type.


Sites without text formats currently allowing the `type` attribute can enable it manually using the "Allow the user to choose a list style type" option from the List CKEditor 5 plugin settings:

---

## [ViewsConfigUpdater is now a service](https://www.drupal.org/node/3530638)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`Drupal\views\ViewsConfigUpdater` is now a service to properly persist the `$deprecationsEnabled` property.


Before:
`\Drupal::classResolver('Drupal\views\ViewsConfigUpdater');`


After:
`\Drupal::service('Drupal\views\ViewsConfigUpdater');`

---

## [Passing NULL as the $elements value in RendererInterface::render() is deprecated](https://www.drupal.org/node/3534020)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Passing `NULL` as the value for the `$elements ` parameter in `RendererInterface::render()` is deprecated in Drupal 11.2.3 and will not be supported in Drupal 12.0.0.


Type declarations will be added to the $element parameter in the method signature in Drupal 12.0.0, so that it will look like this:
`public function render(array &$elements)`


Note that the `$is_root_call` parameter was previously deprecated in 11.2.0 and will be removed in Drupal 12.0.0. See this [change record](https://www.drupal.org/node/3497318).

---

## [Recipe input now accepts environment variables](https://www.drupal.org/node/3524496)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

In addition to `config` and `value`, recipe's input source element now accepts `env`.  If `source` is `env`, there must also be an `env` element, which is the name of an environment variable to return. 


The value will always be returned as a string. If the environment variable is not set, an empty string will be returned.


Example:



```php
name: 'Input from environment variables'
input:
  name:
    data_type: string
    description: The name of the site.
    default:
      # Set the source to look for an environment variable.
      source: env
      # Define the name of the environment variable.
      env: SITE_NAME
  slogan:
    data_type: string
    description: The site slogan.
    default:
      source: env
      env: SITE_SLOGAN
config:
  actions:
    system.site:
      # Use the input in the same way we always have.
      simpleConfigUpdate:
        name: ${name}
        slogan: ${slogan}
```

---

## [Update details element templates to add description ID attributes](https://www.drupal.org/node/3509534)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

Details form elements with descriptions were given an invalid `aria-describedby` attribute whose target did not correspond to any HTML element.  All core templates have been updated to add the `id` to a wrapper around the description.  If you have overridden the details.html.twig template, then you should update your template in order to have valid markup and improve accessibility.


Basic Example
Before:



```php
{{ description }}
```

After:



```php
{%- if description -%}
    {% set description_attributes = create_attribute({id: attributes['aria-describedby']}) %}
    <div{{ description_attributes }}>{{ description }}</div>
  {%- endif -%}
```
Example description with pre-existing wrapper
Before:



```php
{%- if description -%}
      <div class="details-description">{{ description }}</div>
    {%- endif -%}
```

After:



```php
{%- if description -%}
      {% set description_attributes = create_attribute({id: attributes['aria-describedby']}) %}
      <div{{ description_attributes.addClass(['details-description']) }}>{{ description }}</div>
    {%- endif -%}
```

---

## [node_type_get_description is deprecated](https://www.drupal.org/node/3531945)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`node_type_get_description($node_type)` is deprecated, use `$node_type->getDescription()` instead

---

## [HtaccessWriter requires a Settings constructor argument](https://www.drupal.org/node/3249817)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

HtaccessWriter now requires a Settings constructor argument.


Before



```php
file.htaccess_writer:
    class: Drupal\Core\File\HtaccessWriter
    arguments: ['@logger.channel.security', '@stream_wrapper_manager']
```

After



```php
file.htaccess_writer:
    class: Drupal\Core\File\HtaccessWriter
    arguments: ['@logger.channel.security', '@stream_wrapper_manager', '@settings']
```

---

## [Automatic creation of .htaccess files can be disabled](https://www.drupal.org/node/3525119)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

Sites not using the Apache HTTP server or that use a web server configuration that protects writable file directories can disable the creation of the `.htaccess`.


To disable file creation `auto_create_htaccess` must be set to `FALSE` in the site `settings.php` file.


Example
`$settings['auto_create_htaccess'] = FALSE`


If disabled, make sure to follow the guide for [ensuring directories are secure and prevent scripts from being executed](https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal/securing-file-permissions-and-ownership).

---

## [New argument ($persist = TRUE) added to WorkspaceManagerInterface::setActiveWorkspace()](https://www.drupal.org/node/3532912)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new `$persist` boolean argument (`TRUE` by default) has been added to `WorkspaceManagerInterface::setActiveWorkspace()`.


This argument will be added to the method's signature as an API addition starting with Drupal 12.0.

---

## [Calling WorkspaceManager::__construct without an iterable list of workspace negotiators is deprecated](https://www.drupal.org/node/3532939)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The seventh argument for `\Drupal\workspaces\WorkspaceManager::__construct()` must now be an iterable list of workspace negotiators.

---

## [\Drupal\Core\Utility\Error::currentErrorHandler is deprecated](https://www.drupal.org/node/3529500)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

`\Drupal\Core\Utility\Error::currentErrorHandler is deprecated`, use [`get_error_handler()` ](https://www.php.net/manual/en/function.get-error-handler.php)instead.


The `get_error_handler` function is added in PHP 8.5. Until Drupal requires PHP 8.5 a polyfill is provided via [`symfony/polyfill-php85`](https://github.com/symfony/polyfill-php85).

---

## [New BlockContentCreationTrait for tests interacting with block content entities](https://www.drupal.org/node/3532512)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new trait `Drupal\Tests\block_content\Traits\BlockContentCreationTrait` has been added with the following methods: `createBlockContent` and `createBlockContentType`


These methods previously existed on the base class `\Drupal\Tests\block_content\Functional\BlockCotentTestBase`


Tests that create block content types and/or entities can now reuse these functions to reduce duplication.

---

## [status and info settings in block_content blocks are deprecated](https://www.drupal.org/node/3499836)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

The 'status' and 'info settings for block_content blocks are deprecated. They were unused, so there is no replacement

---

## [JSON:API filter constants have been deprecated](https://www.drupal.org/node/3495601)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

The .module file of jsonapi only contains constants. By changing to class constants, we can actually remove the `.module` file in a future update. This requires changing any usage of the constants below to their class equivalent.



- `JSONAPI_FILTER_AMONG_ALL` has been deprecated. Use `\Drupal\jsonapi\JsonApiFilter::AMONG_ALL` instead.

- `JSONAPI_FILTER_AMONG_PUBLISHED` has been deprecated. Use `\Drupal\jsonapi\JsonApiFilter::AMONG_PUBLISHED` instead.

- `JSONAPI_FILTER_AMONG_ENABLED` has been deprecated. Use `\Drupal\jsonapi\JsonApiFilter::AMONG_ENABLED` instead.

- `JSONAPI_FILTER_AMONG_OWN` has been deprecated. Use `\Drupal\jsonapi\JsonApiFilter::AMONG_OWN` instead.

---

## [node_title_list is deprecated](https://www.drupal.org/node/3531959)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

This is a procedural function in node.module that was previously used by statistics and forum modules. It is no longer used by Drupal core.


To replicate the functionality, you can do the following:



```php
$nodes = \Drupal::entityTypeManager()->getStorage('node')->loadMultiple($nids);

    $items = [];
    foreach ($nids as $nid) {
      $node = \Drupal::service('entity.repository')->getTranslationFromContext($nodes[$nid]);
      $item = $node->toLink()->toRenderable();
      $this->renderer->addCacheableDependency($item, $node);
      $items[] = $item;
    }

    return [
      '#theme' => 'item_list__node',
      '#items' => $items,
      '#title' => $title,
      '#cache' => [
        'tags' => \Drupal::entityTypeManager()->getDefinition('node')->getListCacheTags(),
      ],
    ];
```

---

## [Add mergeWith() to AjaxResponse for merging with another response](https://www.drupal.org/node/3486330)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Introduce a new `mergeWith()`method in `core/lib/Drupal/Core/Ajax/AjaxResponse.php`.


A new mergeWith method has been added to the AjaxResponse class. This allows developers to another AjaxResponse into the current one.


Example



```php
$response1 = new AjaxResponse();
$response2 = new AjaxResponse();

$response1->addCommand(new MessageCommand('Your changes have been saved.'));
$response2->addCommand(new MessageCommand('Your changes have been saved but saying it again.'));

/** @var \Drupal\Core\Ajax\AjaxResponse */
return $response1->mergeWith($response2);
```

---

## [system/base split into more conditionally loaded libraries](https://www.drupal.org/node/3530832)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Various CSS files previously included in the system/base library have been split into smaller asset libraries and are now conditionally loaded with the elements or templates that require them.


Themes or modules relying on CSS specified in those libraries may need to include the new library in #attached.


Themes that were overriding the files may need to adjust to override the new libraries.


See [https://www.drupal.org/node/3432346](https://www.drupal.org/node/3432346) for examples of the changes needed.


Libraries moved in 11.3.0:



- [#3512285: Split item-list.module.css out to its own library](https://www.drupal.org/project/drupal/issues/3512285)

- [#3512404: Move reset-appearance.module.css to its own library](https://www.drupal.org/project/drupal/issues/3512404)

---

## [A new database driver (mysqli) for MySQL/MariaDB for parallel queries](https://www.drupal.org/node/3516913)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

A new experimental database driver for MySQL and MariaDB has been added. The new database driver is named MySQLi. The database driver is labeled as `experimental` and therefore not yet fully supported. Use the database driver only for evaluating purposes. The database driver is also marked as hidden.


The new database driver uses another PHP extension to connect to MySQL/MariaDB. The default database driver MySQL/MariaDB uses the mysql PDO PHP extension. The new database driver uses the mysqli PHP extension. This is a more modern PHP extension which allows database queries to be run in parallel instead of sequential as with the PDO extension. For this to work we will use the async Revolt ([https://revolt.run/](https://revolt.run/)) PHP event loop. The creation of this new database driver will unblock work on the use of the async PHP event loop. Examples of how this will make Drupal faster ():
 - The loading of Drupal entities with many fields will be faster. Every field stored its values in its own database table.
 - The loading of views with linked entities will be faster.
 - Any place where multiple queries can run in parallel can be made faster.


How to test this new database driver. The database driver cannot be selected during the install process. Select the regular database driver for MySQL/MariaDB during the site install process or do a `drush site:install`. When you have a running Drupal site, first install/enable the mysqli module, then change in the `settings.php` file the used database connection in the following way.


Install the mysqli module:
`drush pm:install mysqli`


Before:



```php
$databases['default']['default'] = array (
  'database' => 'db',
  'username' => 'db',
  'password' => 'db',
  'host' => 'db', 
  'prefix' => '',
  'driver' => 'mysql',
  'namespace' => 'Drupal\\mysql\\Driver\\Database\\mysql',
  'autoload' => 'core/modules/mysql/src/Driver/Database/mysql/',
);
```

After:



```php
$databases['default']['default'] = array (
  'database' => 'db',
  'username' => 'db',
  'password' => 'db',
  'host' => 'db', 
  'prefix' => '',
  'driver' => 'mysqli',
  'namespace' => 'Drupal\\mysqli\\Driver\\Database\\mysqli',
  'autoload' => 'core/modules/mysqli/src/Driver/Database/mysqli/',
  'dependencies' => array(
    'mysql' => array(
      'namespace' => 'Drupal\\mysql',
      'autoload' => 'core/modules/mysql/src/',
    ),
  ),
);
```

On the status page in the Database section you should see something like: "MySQL via mysqli" or "MariaDB via mysqli".


The long term goal is for this new database driver to replace the existing mysql database driver.

---

## [ConfigurableTrait and ConfigurablePluginBase available to reduce plugin boilerplate](https://www.drupal.org/node/2853355)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A majority of plugins implement `\Drupal\Component\Plugin\ConfigurableInterface`, and each have to provide the getter and setter methods.


Due to the existence of `public function defaultConfiguration();`, they must also handle merging defaults in with explicit configuration.


To reduce the boilerplate, there is now  \Drupal\Core\Plugin\ConfigurablePluginBase class as well as \Drupal\Core\Plugin\ConfigurableTrait.


The trait uses NestedArray::MergeDeepArray to merge the provided configuration with the defaults, which properly merges nested keys while preserving integer keys.


Where possible, a plugin should extend ConfigurablePluginBase to implement ConfigurableInterface. This will properly merge the plugin defaults with the provided configuration in the constructor, and provide the getConfiguration and setConfiguration methods. It will also provide a defaultConfiguration method which returns an empty array.  Plugins should override this method with their own defaults.


For plugins that must extend a different plugin base class (and when the base class itself can not extend ConfigurablePluginBase) a trait is also provided to provide the getConfiguration, setConfiguration, and defaultConfiguration methods provided by the interface. If using the trait instead of the base class, it is important to call ->setConfiguration in the class constructor after calling the parent constructor in order to properly merge the provided configuration with the defaults. see [https://www.drupal.org/node/2852190](https://www.drupal.org/node/2852190) for more detail.


The following plugin base classes have been converted to use the new boilerplate:


ConfigurableActionBase
VariantBase
SelectionPluginBase
LayoutDefault
ImageEffectBase
ConfigurableSearchPluginBase
WorkflowTypeBase


Of those above, All except for ImageEffectBase and WorkflowTypeBase have had their merging logic converted to use mergeDeepArray from either mergeDeep or simple array addition.  These should be functionally equivalent, but may return the array keys in a slightly different order, which might require test updates in contrib.

---

## [Calling \Drupal\Core\Render\Renderer::addCacheableDependency with an object that doesn't implement CacheableDependencyInterface is deprecated](https://www.drupal.org/node/3525389)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Before
Calling `\Drupal\Core\Render\Renderer::addCacheableDependency` with a `$dependency` parameter that did not implement `\Drupal\Core\Cache\CacheableDependencyInterface` would result in the page being uncacheable (max-age 0).


This is rarely what the developer intended. There are other ways to mark a page as uncacheable (e.g setting max-age to 0 explicitly)


After
In 13.0.0, the method `\Drupal\Core\Render\Renderer::addCacheableDependency` will be have a type hint for the `$dependency` argument of `\Drupal\Core\Cache\CacheableDependencyInterface`.



```php
public function addCacheableDependency(array &$elements, CacheableDependencyInterface $dependency) {
```

---

## [Transaction::commitOrRelease() method introduced to explicity commit a transaction](https://www.drupal.org/node/3512006)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

Drupal by default commits a transaction when its Transaction object goes out of scope, during the Transaction object destruction.


This is a pattern that deviates from normal PHP behaviour, where explicit commit is required and without explicit commit an uncompleted transaction is rolled back. Also, recently this behaviour caused problems when the destruction order of objects is not predictable (see [#3405976: Transaction autocommit during shutdown relies on unreliable object destruction order (xdebug 3.3+ enabled)](https://www.drupal.org/project/drupal/issues/3405976)).


A new `Transaction::commitOrRelease()` method is added to the Transaction object, to mean explicit commit/savepoint release.


::commitOrRelease() indicates that the transaction control is returned to the parent level in a nested transaction scenario like Drupal's - e.g. a 'savepoint' transaction object returns control to its parent 'root' transaction (that can still be rolled back entirely if necessary); a 'root' transaction returns control to the database by committing (=persisting changed data) the db transaction, etc.  


Before



```php
try {
      $transaction = $this->connection->startTransaction();
      foreach ($this->insertValues as $insert_values) {
        $stmt->execute($insert_values, $this->queryOptions);
        ...
      }
    }
    catch (\Exception $e) {
      if (isset($transaction)) {
        // One of the INSERTs failed, rollback the whole batch.
        $transaction->rollBack();
      }
      // Rethrow the exception for the calling code.
      throw $e;
    }
```

After



```php
$transaction = $this->connection->startTransaction();
    try {
      foreach ($this->insertValues as $insert_values) {
        $stmt->execute($insert_values, $this->queryOptions);
        ...
      }
      $transaction->commitOrRelease();
    }
    catch (\Exception $e) {
      // One of the INSERTs failed, rollback the whole batch and rethrow the exception for the calling code.
      $transaction->rollBack();
      throw $e;
    }
```

---

## [Olivero's table.css moved to a standalone library and attached only to tables.](https://www.drupal.org/node/3517675)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

The `table.css` file from the Olivero theme has moved from the base global-styling library to a new standalone library `olivero.table`. 


This library is now specifically attached in the following contexts:



- The `table.html.twig` template

- Views rendered with the `views-view-table` template

- Fields of the types: text_with_summary, text, and text_long to support tables added through CKEditor.


Example:
Before



```php
/**
 * Implements hook_preprocess_HOOK().
 */
function olivero_preprocess_field(&$variables): void {
  $rich_field_types = ['text_with_summary', 'text', 'text_long'];

  if (in_array($variables['field_type'], $rich_field_types, TRUE)) {
    $variables['attributes']['class'][] = 'text-content';
  }

  if ($variables['field_type'] == 'image' && $variables['element']['#view_mode'] == 'full' && !$variables["element"]["#is_multiple"] && $variables['field_name'] !== 'user_picture') {
    $variables['attributes']['class'][] = 'wide-content';
  }
}
```

```php
/**
 * Implements hook_preprocess_table().
 */
function olivero_preprocess_table(&$variables): void {
  // Mark the whole table and the first cells if rows are draggable.
  if (!empty($variables['rows'])) {
    $draggable_row_found = FALSE;
    foreach ($variables['rows'] as &$row) {
      /** @var \Drupal\Core\Template\Attribute $row['attributes'] */
      if (!empty($row['attributes']) && $row['attributes']->hasClass('draggable')) {
        if (!$draggable_row_found) {
          $variables['attributes']['class'][] = 'draggable-table';
          $draggable_row_found = TRUE;
        }
      }
    }
  }
}
```

After



```php
/**
 * Implements hook_preprocess_HOOK().
 */
function olivero_preprocess_field(&$variables): void {
  $rich_field_types = ['text_with_summary', 'text', 'text_long'];

  if (in_array($variables['field_type'], $rich_field_types, TRUE)) {
    $variables['attributes']['class'][] = 'text-content';
    $variables['#attached']['library'][] = 'olivero/olivero.table';
  }

  if ($variables['field_type'] == 'image' && $variables['element']['#view_mode'] == 'full' && !$variables["element"]["#is_multiple"] && $variables['field_name'] !== 'user_picture') {
    $variables['attributes']['class'][] = 'wide-content';
  }
}
```

```php
/**
 * Implements hook_preprocess_table().
 */
function olivero_preprocess_table(&$variables): void {
  // Mark the whole table and the first cells if rows are draggable.
  if (!empty($variables['rows'])) {
    $draggable_row_found = FALSE;
    foreach ($variables['rows'] as &$row) {
      /** @var \Drupal\Core\Template\Attribute $row['attributes'] */
      if (!empty($row['attributes']) && $row['attributes']->hasClass('draggable')) {
        if (!$draggable_row_found) {
          $variables['attributes']['class'][] = 'draggable-table';
          $draggable_row_found = TRUE;
        }
      }
    }
  }

  $variables['#attached']['library'][] = 'olivero/olivero.table';
}
```

```php
/**
 * Implements hook_preprocess_HOOK() for views-view-table templates.
 */
function olivero_preprocess_views_view_table(&$variables): void {
  $variables['#attached']['library'][] = 'olivero/olivero.table';
}
```

This change reduces unnecessary CSS on pages that do not contain HTML tables.

---

