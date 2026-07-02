# Drupal Core Change Records for 11.4.0

Total Change Records: **221**

---

## 1. [There is a new Theme extension object. system_region_list() and system_default_region() and region related constants are deprecated](https://www.drupal.org/node/3015925)

- **Node ID**: [3015925](https://www.drupal.org/node/3015925)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3015812](https://www.drupal.org/node/3015812)

**Description:**

There is a new Theme extension object.
This has three new methods:

- listAllRegions

- listVisibleRegions

- getDefaultRegion

REGIONS_VISIBLE and REGIONS_ALL in both system.module and BlockRepositoryInterface have been deprecated with no replacement.

system_region_list and system_default_region have both been deprecated.
Use the following instead.

- \Drupal::service('theme_handler')-&gt;getTheme()-&gt;listAllRegions()

- \Drupal::service('theme_handler')-&gt;getTheme()-&gt;listVisibleRegions()

- \Drupal::service('theme_handler')-&gt;getTheme()-&gt;getDefaultRegion()

All helper methods wrapping the global functions also have been deprecated as well:

- \Drupal\block\BlockListBuilder::systemRegionList() use $this-&gt;themeHandler-&gt;getTheme()-&gt;listAllRegions() or $this-&gt;themeHandler-&gt;getTheme()-&gt;listVisibleRegions()  instead.

- \Drupal\block\Controller\BlockController::getVisibleRegionNames() use $this-&gt;themeHandler-&gt;getTheme()-&gt;listVisibleRegions() instead.

- \Drupal\block_place\Plugin\DisplayVariant\PlaceBlockPageVariant::getVisibleRegionNames() use $this-&gt;themeHandler-&gt;getTheme()-&gt;listVisibleRegions() instead.

Several methods on the ThemeHandler return an array of Theme objects instead of Extension objects. As Theme extends from Extension this is allowed. These methods are:

- \Drupal\Core\Extension\ThemeHandlerInterface::listInfo()

- \Drupal\Core\Extension\ThemeHandlerInterface::rebuildThemeData()

\Drupal\Core\Extension\ThemeHandlerInterface::getTheme() now returns a Theme object.

Breaking changes
\Drupal\Core\Extension\ThemeHandlerInterface::addTheme() now only accepts a Theme object.

---

## 2. [New repository service for filter formats. filter_formats(), filter_formats_reset(), filter_get_formats_by_role(), filter_default_format() & filter_fallback_format() are deprecated](https://www.drupal.org/node/3035368)

- **Node ID**: [3035368](https://www.drupal.org/node/3035368)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2536594](https://www.drupal.org/node/2536594)

**Description:**

There's a new service Drupal\filter\FilterFormatRepositoryInterface accomplishing the functionality of  formats. filter_formats(), filter_formats_reset(), filter_get_formats_by_role(), filter_default_format() & filter_fallback_format() which now are deprecated and will be removed in Drupal 13.

In the following table, the $repository variable is the Drupal\filter\FilterFormatRepositoryInterface service (inject the service where possible)

Before
After

filter_formats()
$repository-&gt;getAllFormats()

filter_formats(User::load(123))
$repository-&gt;getFormatsForAccount(User::load(123))

```php
$id = 'basic_html';
$format = FilterFormat::load($id);
filter_get_roles_by_format($format);
```

FilterFormat::load('basic_html')-&gt;getRoles()

filter_get_formats_by_role('moderator')
$repository-&gt;getFormatsByRole('moderator')

filter_default_format(User::load(123))
$repository-&gt;getDefaultFormat(User::load(123)-&gt;id()

filter_fallback_format()
$repository-&gt;getFallbackFormatId()

filter_formats_reset()

```php
// Inject entity type manager and memory cache if possible
$tags = \Drupal::entityTypeManager()
  ->getDefinition('filter_format')
  ->getListCacheTags();
\Drupal::cache('memory')->invalidateTags($tags);
```

Note: This is rarely if ever needed. The new memory caches are automatically invalidated if formats change and browser tests also reset memory caches on POST requests. Before updating the calls, it is recommended to first test if it works without it.

Other changes:

- The filter_format entity is exposing a new method \Drupal\filter\FilterFormatInterface::getRoles(), which returns a list of roles that are allowed for this text format.

- The constructor \Drupal\filter\FilterFormatListBuilder requires now injecting the new  Drupal\filter\FilterFormatRepositoryInterface service as the 5th parameter.

- Passing 'filter_formats' string to drupal_static_reset() is also deprecated.

---

## 3. [Several functions in locale.compare.inc and LocaleProjectStorageInterface are deprecated](https://www.drupal.org/node/3037033)

- **Node ID**: [3037033](https://www.drupal.org/node/3037033)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3037031](https://www.drupal.org/node/3037031)

**Description:**

Several procedural functions in locale.compare.inc have been deprecated.

- locale_translation_flush_projects()

- locale_translation_build_projects()

- locale_translation_project_list()

- _locale_translation_prepare_project_list()

- locale_translation_default_translation_server()

- locale_translation_check_projects()

- locale_translation_check_projects_local()

The service locale.project and the associated LocaleProjectStorage and LocaleProjectStorageInterface have been deprecated

The functionality has been moved to new services LocaleProjectRepository::class and LocaleProjectChecker::class.

There is a new LocaleTranslatableProject value object to assist with tracking project translations.

Deprecated function
Replacement

locale_translation_flush_projects()
\Drupal::service(LocaleProjectRepository::class)-&gt;deleteAll()

drupal_static_reset('locale_translation_project_list')
$this-&gt;memoryCache-&gt;delete('locale_project_list')

locale_translation_build_projects()
\Drupal::service(LocaleProjectRepository::class)-&gt;buildProjects()

locale_translation_project_list()
No replacement

_locale_translation_prepare_project_list()
No replacement.

locale_translation_default_translation_server()
There is no replacement

locale_translation_check_projects()
\Drupal::service(LocaleProjectChecker::class)-&gt;checkProjects()

locale_translation_check_projects_local()
\Drupal::service(LocaleProjectChecker::class)-&gt;checkProjectsLocal()
**LocaleProjectStorage map:**
There was an extensive amount of duplication of code and very small helper functions that were inlined. The table below identifies which methods were moved and which no longer have a public call.

Deprecated method
Replacement

getMultiple
getMultiple

getAll
getAll

set
set

deleteMultiple
deleteMultiple

deleteAll
deleteAll

resetCache
\Drupal::service('cache.memory')-&gt;delete('locale_get_projects')

get
No replacement

getProjects
No replacement

disableAll
No replacement

delete
No replacement

countProjects
No replacement

setMultiple
No replacement

The alias service and interface do not trigger a deprecation due to: [https://www.drupal.org/project/drupal/issues/3588483](https://www.drupal.org/project/drupal/issues/3588483)

---

## 4. [locale_translation_get_file_history(), locale_translation_update_file_history(), and locale_translation_file_history_delete() have been deprecated](https://www.drupal.org/node/3037162)

- **Node ID**: [3037162](https://www.drupal.org/node/3037162)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3037156](https://www.drupal.org/node/3037156)

**Description:**

- locale_translation_get_file_history()

- locale_translation_update_file_history()

- locale_translation_file_history_delete()

have been deprecated and replaced by a new CurrentImportStateStorage service.

There is a new CurrentImportState value object. This stores:

- project

- langcode

- version

- hash

- timestamp

- last_checked

- type

Before
After

locale_translation_get_file_history()
No direct replacement, use \Drupal::service(CurrentImportStateStorage::class)-&gt;get($project, $langcode) This returns ?CurrentImportState

locale_translation_update_file_history()
\Drupal::service(CurrentImportStateStorage::class)-&gt;update()

locale_translation_file_history_delete()
\Drupal::service(CurrentImportStateStorage::class)-&gt;delete()

drupal_static_reset('locale_translation_get_file_history')
\Drupal::service('cache.memory')-&gt;invalidateTags(['locale_current_import'])

New methods:
\Drupal::service(CurrentImportStateStorage::class)-&gt;updateLastChecked
\Drupal::service(CurrentImportStateStorage::class)-&gt;getOutdatedImports

---

## 5. [views_ui_contextual_links_suppress(), views_ui_contextual_links_suppress_push(), views_ui_contextual_links_suppress_pop() have been deprecated.](https://www.drupal.org/node/3039250)

- **Node ID**: [3039250](https://www.drupal.org/node/3039250)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3039248](https://www.drupal.org/node/3039248)

**Description:**

views_ui_contextual_links_suppress(), views_ui_contextual_links_suppress_push(), views_ui_contextual_links_suppress_pop() are deprecated. The functionality they provided has not worked as intended for a long time, so they will be removed with no replacement.

If you were calling these functions directly, remove the calls.

---

## 6. [The core/modules/views_ui/admin.inc file is deprecated](https://www.drupal.org/node/3040111)

- **Node ID**: [3040111](https://www.drupal.org/node/3040111)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3035340](https://www.drupal.org/node/3035340)

**Description:**

The whole core/modules/views_ui/admin.inc file is deprecated and the procedural functions are moved to:

- \Drupal\views\ViewsFormAjaxHelperTrait

- \Drupal\views\ViewsFormHelperTrait

---

## 7. [Shutdown handler functions are deprecated](https://www.drupal.org/node/3053808)

- **Node ID**: [3053808](https://www.drupal.org/node/3053808)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3053773](https://www.drupal.org/node/3053773)

**Description:**

All shutdown functions are deprecated in favor of a new ShutdownHandler class.

Before
After

Registering custom shutdown function:

```php
use Drupal\Core\Shutdown\ShutdownHandler;

ShutdownHandler::getInstance()->add('custom_shutdown_function');
```

```php
drupal_register_shutdown_function('custom_shutdown_function');
```

Get a list of registered functions:

```php
use Drupal\Core\Shutdown\ShutdownHandler;

$list = ShutdownHandler::getInstance()->get();
```

```php
$list = drupal_register_shutdown_function();
```

Reset list of registered shutdown functions:

```php
use Drupal\Core\Shutdown\ShutdownHandler;

$list = ShutdownHandler::getInstance()->reset();
```

```php
$list = &drupal_register_shutdown_function();
$list = [];
```

Restore list of registered shutdown functions:

```php
use Drupal\Core\Shutdown\ShutdownHandler;

ShutdownHandler::getInstance()->reset($restore_list);
```

```php
$list = &drupal_register_shutdown_function();
$list = $restore_list;
```

(Internal) Call registered shutdown functions:

```php
use Drupal\Core\Shutdown\ShutdownHandler;

ShutdownHandler::getInstance()->shutdown();
```

```php
_drupal_shutdown_function();
```

(Internal) Displays and logs any errors that may happen during shutdown:

```php
use Drupal\Core\Utility\Error;

Error::shutdownExceptionHandler($exception);
```

```php
_drupal_shutdown_function_handle_exception($exception);
```

---

## 8. [New 'resolvable_uri' property is added to link field](https://www.drupal.org/node/3143736)

- **Node ID**: [3143736](https://www.drupal.org/node/3143736)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3066751](https://www.drupal.org/node/3066751)

**Description:**

A new resolvable_uri property is added to the link field which can be used in href attribute.

For example if an entity has a field_link and link field has following value:

```php
$entity->field_link = [
    'uri' => 'internal:/',
    'title'=> 'Home',
    'options' => [
      'fragment' => '#main-content',
    ],
  ];
```

Link field output in JSON:API would be in following format:**Before**

```php
"field_link": [
  {
    "uri": "internal:/",
    "title": "Home",
    "options": [
      'fragment' : '#main-content'
    ]
  }
]
```

This isn't very useful for anything consuming the API because it needs to process these links.**After**

```php
"field_link": [
  {
    "uri": "internal:/",
    "resolvable_uri": "/#main-content"
    "title": "Home",
    "options": [
      'fragment' : '#main-content'
    ]
  }
]
```

Now resolvable_uri property can be used in in href attribute.

Due to this change, a new token for link field would exist [entity:field_link:resolvable_uri].
**API Addition:**
Similarly, if a resolvable_uri property is properly set it will initialize all the link property e.g.:

```php
$entity->field_link->resolvable_uri = 'https://www.drupal.org?test_param=test_value#main-content';
```

Same as:

```php
$entity->field_link->uri = 'https://www.drupal.org'
   $entity->field_link->options = [
     'query' => [
       'test_param' => 'test_value',
     ],
     'fragment' => '#main-content',
   ];
```

---

## 9. [Third party can alter the layout builder section render array](https://www.drupal.org/node/3210520)

- **Node ID**: [3210520](https://www.drupal.org/node/3210520)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3062862](https://www.drupal.org/node/3062862)

**Description:**

The entire section render array can be altered using two new hooks, hook_layout_builder_section_regions_render_array_alter and hook_layout_builder_section_render_array_alter

hook_layout_builder_section_regions_render_array_alter
Invoked while the regions array is being assembled, before the layout plugin renders it. The hook receives $regions by reference, along with the Section object and a $context array containing the available contexts and the in_preview flag.

**hook_layout_builder_section_render_array_alter:**

Invoked after the layout plugin has assembled the regions into the final section render array. The hook receives $build by reference, along with the same Section object and $context array.

**Example:**

src/Hook/MyModuleHooks.php:

```php

---

## 10. [Introducing Batch Processor service](https://www.drupal.org/node/3229844)

- **Node ID**: [3229844](https://www.drupal.org/node/3229844)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#2875151](https://www.drupal.org/node/2875151)

**Description:**

**What's changing?:**

- Implemented the initial batch processor service

- Deprecated related procedural Batch API functions

**Batch Processor Instance:**

How to get the instance of the batch processor:

- By using DI and autowire, set the type of the parameter in the class to \Drupal\Core\Batch\BatchProcessor**For example:

```php
use \Drupal\Core\Batch\BatchProcessor;
  // ...
  public function __construct(
    protected readonly BatchProcessor $batchProcessor,
  ) {}
```

- By using in a static or procedural context, use service name equal to \Drupal\Core\Batch\BatchProcessor::class
For example:

```php
use \Drupal\Core\Batch\BatchProcessor;
  // ...
  $batch_processor = \Drupal::service(BatchProcessor::class);
```

**Deprecated Public API:**

Here are the examples of how to replace the procedural public API with calls to the Batch Processor instance:

Deprecated
Replacement

batch_set($batch_builder-&gt;toArray());
$batch_processor-&gt;queue($batch_builder-&gt;toArray());

batch_process();
$batch_processor-&gt;process();

$batch = &batch_get();
$batch = &$batch_processor-&gt;getCurrentBatch();

**Deprecated Internal API:**

Note 1**: The Internal Procedural Batch API is also BC compatible. Internal procedural functions will continue to work as expected until the Drupal 13.0.0 release.**Note 2**: Follow the details about [BC Policy](https://www.drupal.org/about/core/policies/core-change-policies/bc-policy) regarding the usage of the Internal API
Here are the examples of how to replace the procedural internal API with calls to the Batch Processor instance:

Deprecated
Replacement

_batch_needs_update();
$batch_processor-&gt;needsUpdate();

_batch_process();
$batch_processor-&gt;processQueue();

$set = &_batch_current_set();
$set = &$batch_processor-&gt;getCurrentSet();

_batch_next_set();
$batch_processor-&gt;nextSet();

_batch_finished();
$batch_processor-&gt;finishedProcessing();

_batch_shutdown();
$batch_processor-&gt;shutdown();

_batch_append_set($batch, $batch_set);
$batch_processor-&gt;appendSet($batch, $batch_set);

_batch_queue($batch_set);
$batch_processor-&gt;getQueue($batch_set);

---

## 11. [Render control functions hide() and show() are deprecated](https://www.drupal.org/node/3261271)

- **Node ID**: [3261271](https://www.drupal.org/node/3261271)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2258355](https://www.drupal.org/node/2258355)

**Description:**

Instead of using hide() and show() functions, use inline #printed flag manipulation to control render array output:

Before
Now

```php
hide($element);
```

```php
$element['#printed'] = TRUE;
```

```php
show($element);
```

```php
$element['#printed'] = FALSE;
```

---

## 12. [hasRole() has moved from UserInterface to AccountInterface](https://www.drupal.org/node/3283218)

- **Node ID**: [3283218](https://www.drupal.org/node/3283218)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3228209](https://www.drupal.org/node/3228209)

**Description:**

[#2991232: Add hasRole() method to AccountProxy and UserSession classes](/project/drupal/issues/2991232) introduced a hasRole() method.  Due to backwards compatibility concerns, it was added directly to the Drupal\Core\Session\AccountProxy and Drupal\Core\Session\UserSession classes.

This method is now defined in Drupal\Core\Session\AccountInterface, which all the relevant classes implement and which Drupal\user\UserInterface extends.

If you have code that implements AccountInterface, you may need to add a hasRole() method if you are not extending a class that already provides it.

---

## 13. [The '#url' property in the responsive_image_formatter theme element is now a Url object](https://www.drupal.org/node/3291487)

- **Node ID**: [3291487](https://www.drupal.org/node/3291487)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3064751](https://www.drupal.org/node/3064751)

**Description:**

The #url variable in the responsive image theme element is now a Drupal\Core\Urlobject, which allows for manipulating URL options. Previously the variable was a string.

This variable is available as url in the response-image-formatter.html.twig template.

**Before:**

```php
#[Hook('preprocess_responsive_image_formatter')]
  public function preprocessResponsiveImageFormatter(array &$variables): void {
    $url = $variables['#url'];
    // E.g., $url === 'https://drupal.org'.
  }
```

**After:**

```php
#[Hook('preprocess_responsive_image_formatter')]
  public function preprocessResponsiveImageFormatter(array &$variables): void {
    $url = $variables['#url'];
    // E.g., $url->toString() === 'https://drupal.org'.
  }
}
```

**Impact on themes and modules:**

Any theme or module that implements a preprocess_response_image_formatter hook that uses the #url property will need to be updated to account for the new variable type.

Use in Twig templates is generally not affected, as {{ url }} outputs the URL string as before.

---

## 14. [RowPluginBase::render() only accepts \Drupal\views\ResultRow](https://www.drupal.org/node/3309958)

- **Node ID**: [3309958](https://www.drupal.org/node/3309958)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3041170](https://www.drupal.org/node/3041170)

**Description:**

RowPluginBase::render() will no longer accept arguments other than instances of \Drupal\views\ResultRow.

This will be type-hinted in Drupal 12.0.

---

## 15. [\Drupal\Core\Field\Plugin\Field\FieldFormatter\EntityReferenceEntityFormatter::RECURSIVE_RENDER_LIMIT and ::$recursiveRenderDepth are deprecated](https://www.drupal.org/node/3316878)

- **Node ID**: [3316878](https://www.drupal.org/node/3316878)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2940605](https://www.drupal.org/node/2940605)

**Description:**

Both \Drupal\Core\Field\Plugin\Field\FieldFormatter\EntityReferenceEntityFormatter::RECURSIVE_RENDER_LIMIT and \Drupal\Core\Field\Plugin\Field\FieldFormatter\EntityReferenceEntityFormatter::$recursiveRenderDepth are deprecated.

A referenced entity can now be successfully re-rendered more than 20 times. Example use cases:

- Running batch search indexing on rendered content with the same media referenced by multiple indexed nodes.

- Rendering a node multiple times in a batch to be sent out as email newsletters.

Rendering recursion is now protected by tracking whether an entity is currently being rendered. This is done in Drupal\Core\Entity\EntityViewBuilder, where #pre_render and #post_render callbacks are added to the entity render array in getBuildDefaults(). These callbacks set and unset a tracking array entry indexed by the imploded cache keys of the render array for the entity.

---

## 16. [InstallerRouteBuilder is no longer needed](https://www.drupal.org/node/3324749)

- **Node ID**: [3324749](https://www.drupal.org/node/3324749)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3311365](https://www.drupal.org/node/3311365)

**Description:**

Due to the changes to add attribute based route discovery the \Drupal\Core\Installer\InstallerRouteBuilder is no longer needed. There is no replacement.

---

## 17. [RouteBuilder no longer needs the module handler and controller resolver injected](https://www.drupal.org/node/3324751)

- **Node ID**: [3324751](https://www.drupal.org/node/3324751)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3311365](https://www.drupal.org/node/3311365)

**Description:**

Due to the changes to add attribute based route discovery the \Drupal\Core\Routing\RouteBuilder no longer needs the module handler and controller resolver injected

---

## 18. [PHP Attributes can be used for route definition and discovery](https://www.drupal.org/node/3324758)

- **Node ID**: [3324758](https://www.drupal.org/node/3324758)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3311365](https://www.drupal.org/node/3311365)

**Description:**

Starting with Drupal 11.4.0, you can now use attributes on your controllers to specify the routes the controller is used for. Any classes in a module's Controller namespace (for example, Drupal\example\Controller) that have the Symfony\Component\Routing\Attribute\Route attribute will be picked up as route definitions.

This supplements the existing .routing.yml based declarations.

The structure of the properties remains the same, but human-readable strings need to be explicitly translated with \Drupal\Core\StringTranslation\TranslatableMarkup as in the example below.

**Example using .yml file:**

```php
system.401:
  path: '/system/401'
  defaults:
    _controller: '\Drupal\system\Controller\Http4xxController:on401'
    _title: 'Unauthorized'
  requirements:
    _access: 'TRUE'
```

**Example using a route attribute:**

```php
namespace Drupal\system\Controller;

use Drupal\Core\StringTranslation\TranslatableMarkup;
use Symfony\Component\Routing\Attribute\Route;

class Http4xxController extends ControllerBase {

 /**
- The default 401 content.
- * @return array
- A render array containing the message to display for 401 pages.
   */
  #[Route(
    path: '/system/401',
    name: 'system.401',
    requirements: ['_access' => 'TRUE'],
    defaults: ['_title' => new TranslatableMarkup('Unauthorized')]
  )]
  public function on401() {
    return [
      '#markup' => $this->t('Log in to access this page.'),
    ];
  }

}
```

**Multiple routes in one class:**

The #[Route] attribute can also be applied at class level to provide defaults. The name attribute will be used to prefix all routes in the class. Other attributes will be merged together. The following is equivalent to the above example:

```php
namespace Drupal\system\Controller;

use Drupal\Core\StringTranslation\TranslatableMarkup;
use Symfony\Component\Routing\Attribute\Route;

#[Route(
  name: 'system.',
  requirements: ['_access' => 'TRUE'],
)]
class Http4xxController extends ControllerBase {

 /**
- The default 401 content.
- * @return array
- A render array containing the message to display for 401 pages.
   */
  #[Route(
    path: '/system/401',
    name: '401',
    defaults: ['_title' => new TranslatableMarkup('Unauthorized')]
  )]
  public function on401() {
    return [
      '#markup' => $this->t('Log in to access this page.'),
    ];
  }

}
```

**Using class and method level attributes:**

Defaults can be set at class level, and extended or overridden at method level as follows:

The name and path properties are concatenated, so where needed a prefix can be specified at class level with individual suffixes at method level.

defaults, requirements and options keys are merged together, with method level taking precedence.

schemes and methods arrays are merged together.

host and priority values take precedence from method level, but defaults can be set at class level.

**Routes using __invoke:**

The class-level route can also be used to define a route using the __invoke method:

```php
namespace Drupal\router_test\Controller;

use Drupal\Core\Controller\ControllerBase;
use Symfony\Component\Routing\Attribute\Route;

/**
- Test controller.
 */
#[Route(
  '/test_class_attribute',
  'test_class_attribute',
  requirements: ['_access' => 'TRUE']
)]
class TestClassAttribute extends ControllerBase {

  /**
- Provides test content.
   */
  public function __invoke() {
    return ['#markup' => 'Testing __invoke() with a Route attribute on the class'];
  }

}
```

---

## 19. [Entity bundles can be defined in a method on the entity class](https://www.drupal.org/node/3333249)

- **Node ID**: [3333249](https://www.drupal.org/node/3333249)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3112239](https://www.drupal.org/node/3112239)

**Description:**

Entity bundles can now be defined in code with a static method on the entity class, in addition to using hook_entity_bundle_info() or hook_entity_bundle_info_alter().

This follows the same pattern as using entity class methods to define base fields (baseFieldDefinitions()) and bundle fields (bundleFieldDefinitions()).

The format of the return value of bundleDefinitions() is the same as for the entity bundle info hooks:

```php
/**
- {@inheritdoc}
   */
  public static function bundleDefinitions(EntityTypeInterface $entity_type) {
    return [
      'my_bundle' => [
        'label' => t('My bundle label'),
      ],
    ];
  }
```

---

## 20. [The queue backend definition for the queue worker in settings.php file is deprecated](https://www.drupal.org/node/3338022)

- **Node ID**: [3338022](https://www.drupal.org/node/3338022)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#2821989](https://www.drupal.org/node/2821989)

**Description:**

**Before:**

```php
// ...
// Content of settings.php file.
$settings['queue_service_' . $queue_worker_id] = \Drupal\Core\Queue\QueueDatabaseFactory::class;
//...
```

**After:**

```php
// ...
// Content of queue worker plugin file.

/**
- Process a queue of media items to fetch their thumbnails.
 */
#[QueueWorker(
  id: 'media_entity_thumbnail',
  title: new TranslatableMarkup('Thumbnail downloader'),
  cron: ['time' => 60],
  queue_service: \Drupal\Core\Queue\QueueDatabaseFactory::class,
)]
class ThumbnailDownloader extends QueueWorkerBase implements ContainerFactoryPluginInterface {
// ...
```

**Altering capabilities****Now, as the service name is part of the queue worker plugin definition information, it can be altered via the queue plugin manager hook_info:

```php
#[Hook('queue_info_alter')]
public static function queueInfoAlter(array &$queues): void {
  $queues['media_entity_thumbnail']['queue_service'] = \Drupal\Core\Queue\QueueDatabaseFactory::class;
}
```

Changes in the class definition:**
Class Drupal\Core\Queue\QueueFactory now takes the QueueWorkerManagerInterface dependency in the constructor as a new argument.

---

## 21. [SysLog config property replaced with config.factory service closure](https://www.drupal.org/node/3343843)

- **Node ID**: [3343843](https://www.drupal.org/node/3343843)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3103620](https://www.drupal.org/node/3103620)

**Description:**

The Drupal\syslog\Logger\SysLog class now uses a service closure to lazily load the config.factory service.  This works around certain cases of circular references that appear if a config storage dependency (such as an alternative cache backend) requires logging, and a logger depends on config storage.  Subclasses should retrieve the configuration from ($this-&gt;configFactory)()-&gt;get('syslog.settings') rather than $this-&gt;config.

---

## 22. [The trusted data concept in Config and Config Entities is deprecated](https://www.drupal.org/node/3348180)

- **Node ID**: [3348180](https://www.drupal.org/node/3348180)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3347842](https://www.drupal.org/node/3347842)

**Description:**

The concept of "trusted data" in the configuration system has been deprecated in Drupal 11.4.0 and will be removed in Drupal 13.0.0. Previously, code could mark configuration data as "trusted" to skip schema-based validation and casting on save, which was primarily used during module/theme installation for performance. Configuration schema is now always used to validate, cast, and sort configuration data on save.

**What changed:**

The following methods and parameters are deprecated:

- ConfigEntityInterface::trustData() — deprecated, no replacement.

- The $has_trusted_data parameter on Config::save() — deprecated. Call save() with no arguments.

Configuration data is now always cast against schema on every save. This ensures that:

- Configuration values are always correctly typed according to their schema.

- Configuration map keys are always sorted according to their schema definition order.

**How to update your code:**

**Remove calls to trustData():**

**Before:**

```php
$entity->trustData();
  $entity->save();
```

**After:**

```php
$entity->save();
```

**Remove the $has_trusted_data argument from Config::save():**

**Before:**

```php
$config = \Drupal::configFactory()->getEditable('mymodule.settings');
  $config->setData($data);
  $config->save(TRUE);
```

**After:**

```php
$config = \Drupal::configFactory()->getEditable('mymodule.settings');
  $config->setData($data);
  $config->save();
```

**Manipulating configuration without schema in hook_update_N():**

If you need to write configuration data in a hook_update_N() that does not conform to the current schema — for example, when you are migrating data between configuration keys, removing keys, or making changes where the schema has not yet been updated — use the config.storage service to read and write raw configuration data directly, bypassing the configuration factory and schema casting entirely.

**Before (using trusted data to skip schema):**

```php
function mymodule_update_11401() {
    $config = \Drupal::configFactory()->getEditable('mymodule.settings');
    // Manipulate data that doesn't match current schema...
    $config->set('new_key', $config->get('old_key'));
    $config->clear('old_key');
    $config->save(TRUE);
  }
```

**After (using config.storage service):**

```php
function mymodule_update_11401() {
    /** @var \Drupal\Core\Config\StorageInterface $config_storage */
    $config_storage = \Drupal::service('config.storage');
    $data = $config_storage->read('mymodule.settings');
    if ($data !== FALSE) {
      // Manipulate raw data without schema validation.
      $data['new_key'] = $data['old_key'];
      unset($data['old_key']);
      $config_storage->write('mymodule.settings', $data);
    }
  }
```

**Manipulating configuration without schema in tests:**

It can be useful to write config that does not comply with schema in order to test updates.

**Before (using trusted data to skip schema):**

```php
// Explicitly set the `items_per_page` setting to a string without casting.
     $block->set('settings', [
       'items_per_page' => '5',
    ])->trustData()->save();
```

**After (using RawConfigWriterTrait):**
Use the Drupal\Tests\Traits\Core\Config\RawConfigWriterTrait trait in use test class.

```php
// Explicitly set the `items_per_page` setting to a string without casting.
     $block->set('settings', [
       'items_per_page' => '5',
      'label' => 'Test',
    ]);
    $this->writeRawConfigEntity($block);
```

There is also a method on the trait to manipulate simple config  - writeRawConfig()

---

## 23. [Link field widget supports route:{$route_name}](https://www.drupal.org/node/3367114)

- **Node ID**: [3367114](https://www.drupal.org/node/3367114)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3115188](https://www.drupal.org/node/3115188)

**Description:**

Previously

route:view.frontpage.page_1 entered in link widget after save would strip the route: on a resave.

Now

route:view.frontpage.page_1 entered in link widget after save will not strip the route: on a resave.

---

## 24. [views_add_contextual_links() has been deprecated](https://www.drupal.org/node/3382344)

- **Node ID**: [3382344](https://www.drupal.org/node/3382344)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2571679](https://www.drupal.org/node/2571679)

**Description:**

The  views_add_contextual_links() function has been deprecated.

Use Drupal\views\ContextualLinks::addLinks() instead.

**Before:**

```php
views_add_contextual_links($element, 'view', $view->current_display);
```

**After:**

```php
use Drupal\views\ContextualLinksHelper;
...
\Drupal::service(ContextualLinksHelper::class)->addLinks($element, 'view', $view->current_display);
```

---

## 25. [JSON data type added to database schema API](https://www.drupal.org/node/3389682)

- **Node ID**: [3389682](https://www.drupal.org/node/3389682)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3343634](https://www.drupal.org/node/3343634)

**Description:**

Drupal's Schema and Database API now support JSON as a native column type, with a driver-neutral query API for filtering by values at a JSON path.

**The json field type:**

Declare a JSON column in hook_schema() by setting its type to json. Each driver maps to its native JSON type: JSON on MySQL/MariaDB and SQLite, jsonb on PostgreSQL.

```php
$schema['my_table'] = [
  'fields' => [
    'id' => ['type' => 'serial', 'not null' => TRUE],
    'data' => ['type' => 'json'],
  ],
  'primary key' => ['id'],
];
```

**Querying JSON values:**

Use Select::jsonCondition() (or its trait counterpart QueryConditionTrait::jsonCondition()) to filter a select query by the value at a JSON path:

```php
$query = $connection->select('my_table', 'm');
$query->addField('m', 'id');
$query->jsonCondition('m.data', '$.status', 'published');
$ids = $query->execute()->fetchCol();
```

Supported operators cover the usual equality and comparison operators (=, &lt;&gt;, &lt;, &lt;=, &gt;, &gt;=), IN / NOT IN, BETWEEN / NOT BETWEEN, IS NULL / IS NOT NULL, HAS KEY, JSON_CONTAINS, and the string-matching CONTAINS, STARTS_WITH, and ENDS_WITH.

For more complex cases, get a JsonExpression directly and embed it in custom SQL:

```php
$expression = $connection->jsonExpression('data', '$.number');
$query->addExpression($expression->toCastSql(JsonCastType::Int), 'n');
$query->orderBy('n');
```

**For contrib database drivers:**

Drivers must provide a Drupal\Core\Database\Query\JsonExpression subclass under their own namespace to support JSON queries. Connection::jsonExpression() picks it up automatically via getDriverClass('JsonExpression').

Until Drupal 12.0.0 the base Connection::jsonExpression() is non-abstract and throws a \LogicException (after an E_USER_DEPRECATED) when called on a driver that has no implementation. From Drupal 12.0.0 the method becomes abstract. See the mysql, pgsql, and sqlite subclasses in core for working examples.

---

## 26. [\Drupal\Core\Menu\LocalTaskManager constructor changes](https://www.drupal.org/node/3443775)

- **Node ID**: [3443775](https://www.drupal.org/node/3443775)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3090668](https://www.drupal.org/node/3090668)

**Description:**

Calling \Drupal\Core\Menu\LocalTaskManager() without \Psr\Log\LoggerInterface $logger argument is deprecated.

\Drupal\Core\Menu\LocalTaskManager()::__construct() has been changed:

- Add a \Psr\Log\LoggerInterface compatible logger like @logger.channel.default service which is also used in the @plugin.manager.menu.local_task service

---

## 27. [Drupal\user\ToolbarLinkBuilder requires a new parameter](https://www.drupal.org/node/3455774)

- **Node ID**: [3455774](https://www.drupal.org/node/3455774)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2313309](https://www.drupal.org/node/2313309)

**Description:**

\Drupal\user\ToolbarLinkBuilder requires now a new parameter of type \Drupal\Core\Extension\ModuleHandlerInterface. Custom modules extending ToolbarLinkBuilder should adjust the code accordingly.

---

## 28. [New language.admin_language_render service](https://www.drupal.org/node/3456217)

- **Node ID**: [3456217](https://www.drupal.org/node/3456217)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2313309](https://www.drupal.org/node/2313309)

**Description:**

A new service has been added ('language.admin_language_render') to enable rendering elements in the admin language.

This service contains the method public static function applyTo(array $type): array  which adds the render callbacks to a render element.

Code example:

```php
$element = AdminLanguageRender::applyTo($element);
```

---

## 29. [W3C compliant testing](https://www.drupal.org/node/3460567)

- **Node ID**: [3460567](https://www.drupal.org/node/3460567)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3421202](https://www.drupal.org/node/3421202), [#3462682](https://www.drupal.org/node/3462682)

**Description:**

**Test configuration changes:**

Drupal core now tests using selenium/standalone-chrome:latest via a W3C compliant webdriver.

To enable W3C mode in WebDriverTestBase tests (FunctionalJavascript) the MINK_DRIVER_ARGS_WEBDRIVER environment variable must pass "w3c": true in the options. For example:
["chrome", {"browserName":"chrome", "goog:chromeOptions":{"w3c":true, "args":["--no-sandbox","--ignore-certificate-errors", "--allow-insecure-localhost", "--headless", "--dns-prefetch-disable"]}}, "http://selenium:4444"]
To enable W3C mode in Nightwatch tests, the DRUPAL_TEST_WEBDRIVER_W3C environment variable must to be set to true. For example:
Add DRUPAL_TEST_WEBDRIVER_W3C=true to your .env file.
Non-W3C compliant testing is deprecated and will be removed in Drupal 12.

**How to make your tests W3C webdriver compatible:**

- Replace calls to ::moveTo() with ::mouseOver()

**Before:**

```php
$driver_session = $this->getSession()->getDriver()->getWebDriverSession();
    $element = $driver_session->element('css selector', '#block-branding');
    $driver_session->moveto(['element' => $element->getID()]);
```

**After:**

```php
$this->getSession()->getDriver()->mouseOver('.//*[@id="block-branding"]');
```

- Replace calls to ::getAlert_text() with alert()-&gt;getText()

**Before:**

```php
$session->getDriver()->getWebDriverSession()->getAlert_text();
```

**After:**

```php
$session->getDriver()->getWebDriverSession()->alert()->getText();
```

- Replace calls to ::accept_alert() with alert()-&gt;accept()

**Before:**

```php
$session->getDriver()->getWebDriverSession()->accept_alert();
```

**After:**

```php
$session->getDriver()->getWebDriverSession()->alert()-> accept();
```

- If you used the method location() on a \WebDriver\Element instance only for getting the x and y position of a html element, replace calls to ::location() with ::rect()

**Before:**

```php
$session->getDriver()->getWebDriverSession()->element('xpath', $xpath)->location();
```

**After:**

```php
$session->getDriver()->getWebDriverSession()->element('xpath', $xpath)->rect();
```

---

## 30. [Calling \Drupal\Core\Menu\MenuLinkTree::__construct() without the event dispatcher is deprecated](https://www.drupal.org/node/3463907)

- **Node ID**: [3463907](https://www.drupal.org/node/3463907)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3091246](https://www.drupal.org/node/3091246)

**Description:**

The menu.link_tree (\Drupal\Core\Menu\MenuLinkTree) service now requires the event dispatcher service to be injected via the constructor.

Pass the event_dispatcher service to \Drupal\Core\Menu\MenuLinkTree::__construct() if you instantiate it.

---

## 31. [Layout plugin definitions require the label to be set](https://www.drupal.org/node/3464076)

- **Node ID**: [3464076](https://www.drupal.org/node/3464076)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3463592](https://www.drupal.org/node/3463592)

**Description:**

Layout plugin definitions require a label to be set, either directly through the plugin label property, or through a deriver. Having the label set in the plugin definition will enforced in Drupal 12.0.0.

The layout plugin category key is also required.  If the category is missing, then one will be assigned automatically by LayoutPluginManager equal to the plugin provider's name.

---

## 32. [Added symfony/polyfill-php86](https://www.drupal.org/node/3464803)

- **Node ID**: [3464803](https://www.drupal.org/node/3464803)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3590237](https://www.drupal.org/node/3590237)

**Description:**

With the symfony/polyfill-php86 component added, the following features added to PHP 8.6 core are available in Drupal 11.4:

- [clamp()](https://wiki.php.net/rfc/clamp_v2)

- ARRAY_FILTER_USE_VALUE constant

- [SortDirection enum](https://wiki.php.net/rfc/sort_direction_enum)

See [https://github.com/symfony/polyfill-php86](https://github.com/symfony/polyfill-php86) for more.

---

## 33. [Touch events detection is deprecated](https://www.drupal.org/node/3467337)

- **Node ID**: [3467337](https://www.drupal.org/node/3467337)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3375181](https://www.drupal.org/node/3375181)

**Description:**

The core/drupal.touchevents-test library is deprecated and core will no longer add the touchevents-enabled and touchevents-disabled classes to pages.

Instead, native CSS media queries can be used:

```php
@media (pointer: fine)

  @media (pointer: coarse)
```

[https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@med...](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/pointer)

---

## 34. [Core library definitions support before/after ordering](https://www.drupal.org/node/3467489)

- **Node ID**: [3467489](https://www.drupal.org/node/3467489)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#1945262](https://www.drupal.org/node/1945262)

**Description:**

Where a library needs to be loaded explicitly before or after another library, regardless of dependencies, the new 'before' and 'after' keys can be used, e.g.:

```php
before_main:
  js:
    before_main.js: {}
  before:
    - common_test/main

main:
  js:
    main.js: {}

after_main:
  js:
    after_main.js: {}
  after:
    - common_test/main
```

By using this, library order can be modified without having to force a dependency, which can lead to unnecessary library loading if used purely to affect loading order.
Custom weights for individual files are now discouraged in favour of this new method, and may be deprecated in a future release.

---

## 35. [JSON:API no longer validates every response against schema by default](https://www.drupal.org/node/3478687)

- **Node ID**: [3478687](https://www.drupal.org/node/3478687)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3472008](https://www.drupal.org/node/3472008)

**Description:**

JSON:API had a soft dependency on justinrainbow/json-schema; validation of the response was performed automatically if the dependency was enabled via \Drupal\jsonapi\EventSubscriber\ResourceResponseValidator. This has a sizable performance hit.

In Drupal 11.4 and 12.0 this dependency will be shipped as a runtime dependency in order to assist SDC and Canvas with additional validation.

JSON:API is considered stable and has an extensive suite of tests. As a result, JSON:API validation has been moved to a test-only module. If your tests were relying on this validation, you can re-enable it by installing the jsonapi_response_validator module.

---

## 36. [Overriding libraries is now consistent even if already overridden by a parent theme](https://www.drupal.org/node/3489303)

- **Node ID**: [3489303](https://www.drupal.org/node/3489303)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#2642122](https://www.drupal.org/node/2642122)

**Description:**

If a parent theme overrode a grandparent theme, the theme libraries overrides would no longer work the same and a non-obvious workaround was needed. Now a library override of a grandparent theme is done in the same way as the first override.

For example, Olivero's olivero.info.yml has:

```php
libraries-override:
  core/drupal.ajax:
    css:
      component:
        core/components/ajax-progress.module.css: css/components/ajax-progress.module.css
```

If a sub-theme of Olivero is created, let's call it Verona can now do this in verona.info.yml:

```php
libraries-override:
  core/drupal.ajax:
    css:
      component:
        core/components/ajax-progress.module.css: false
```

Previously developers would need to search where the css was overridden and hard-code the path like:

```php
libraries-override:
  core/drupal.ajax:
    css:
      component:
        /core/themes/olivero/css/components/ajax-progress.module.css: false
```

---

## 37. [New service for purging field data](https://www.drupal.org/node/3494023)

- **Node ID**: [3494023](https://www.drupal.org/node/3494023)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2907780](https://www.drupal.org/node/2907780)

**Description:**

A new service has been added for purging field data: \Drupal\Core\Field\FieldPurger, and the following functions have been deprecated:

- field_purge_batch()

- field_purge_field()

- field_purge_field_storage()

**Before:**

```php
field_purge_batch(10);
```

**After:**

```php
\Drupal::service(FieldPurger::class)->purgeBatch(10);
```

**Deprecated without replacement:**

```php
field_purge_field($field_definition);
field_purge_field_storage($field_storage_definition);
```

**Configuration****The field.field_settings:field_purge_batch_size configuration has been deprecated.
Before****field.field_settings:field_purge_batch_size
After**
$settings['field_purge_batch_size'] = '50';

---

## 38. [Kernel tests can make HTTP requests with drupalGet()](https://www.drupal.org/node/3502609)

- **Node ID**: [3502609](https://www.drupal.org/node/3502609)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3390193](https://www.drupal.org/node/3390193), [#3582769](https://www.drupal.org/node/3582769)

**Description:**

Kernel tests can now make HTTP requests and make assertions against the returned content, similarly to Browser tests.

This means that many Browser tests can be converted to Kernel tests, which will run significantly faster.

The test trait \Drupal\Tests\HttpKernelUiHelperTrait provides methods drupalGet(), clickLink(), and assertSession() which work the same way as the corresponding methods in the test trait Drupal\Tests\UiHelperTrait for Browser tests.

After a drupalGet() call, assertSession() can be used to make assertions on the page content using Mink.

The Mink driver passes requests to Drupal's HTTP kernel, rather than simulating a browser.

**Example:**

```php
use Drupal\KernelTests\KernelTestBase;

class MyTest extends KernelTestBase {

  public function testFoo(): void {
    $this->drupalGet('my-path');
    $this->assertSession()->pageTextContains('Page title');
    $this->assertSession()->linkExists('Read more');
    $this->clickLink('Read more');
    $this->assertSession()->pageTextContains('New page title');
  }
```

When converting a Functional test to a Kernel test, the setup tasks that are performed by modules enabled in the test need to be performed with API calls in the test's setup. See [the documentation page on Setup tasks in Kernel tests for more details](https://www.drupal.org/docs/automated-testing/phpunit-in-drupal/setup-tasks-in-kernel-tests).

**Limitations:**

Some things do not work the same as in Browser tests, or will need additional setup tasks:

- Forms cannot be submitted. This functionality may be added in a future issue.

- Session semantics differ from normal page requests. Do not rely on session state beyond its existence (no persistence or regeneration).

- There is no logged in user. Use \Drupal\Tests\user\Traits\UserCreationTrait to set a current user.

- There is no active theme. To place blocks, a test must first install a theme and then set it as active.

- Additional steps are needed for page caching modules to function. See Drupal\Tests\Traits\Core\Cache\PageCachePolicyTrait for details.

---

## 39. [\Drupal\layout_builder\EventSubscriber\SetInlineBlockDependency constructor now requires new parameters](https://www.drupal.org/node/3507600)

- **Node ID**: [3507600](https://www.drupal.org/node/3507600)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3047022](https://www.drupal.org/node/3047022)

**Description:**

Creating a new \Drupal\layout_builder\EventSubscriber\SetInlineBlockDependency now requires new parameters.

entity_type.manager has been replaced by entity.repository and a 5th parameter has been added for current_route_match
**Before:**

```php
new SetInlineBlockDependency(
  \Drupal::service('entity_type.manager'),
  \Drupal::service('database'),
  \Drupal::service('inline_block.usage'),
  \Drupal::service('plugin.manager.layout_builder.section_storage'),
);
```

**After:**

```php
new SetInlineBlockDependency(
  \Drupal::service('entity.repository'),
  \Drupal::service('database'),
  \Drupal::service('inline_block.usage'),
  \Drupal::service('plugin.manager.layout_builder.section_storage'),
  \Drupal::service('current_route_match'),
);
```

---

## 40. [Classes using getFormula() in Views must implement ArgumentInterface](https://www.drupal.org/node/3508630)

- **Node ID**: [3508630](https://www.drupal.org/node/3508630)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3414108](https://www.drupal.org/node/3414108)

**Description:**

Classes implementing getFormula() in views, must implement \Drupal\views\Plugin\views\argument\ArgumentInterface.
**Before:**

```php
class Foo extends ArgumentPluginBase {

  public function getFormula() {
    // Do something.
  }

}
```

**After:**

```php
class Foo extends ArgumentPluginBase implements ArgumentInterface {

  public function getFormula(): string {
    // Do something.
  }

}
```

---

## 41. [Views table alignment style options now relies on core alignment classes](https://www.drupal.org/node/3515029)

- **Node ID**: [3515029](https://www.drupal.org/node/3515029)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3436855](https://www.drupal.org/node/3436855)

**Description:**

Views previously provided its own views-align-* CSS declarations to handle column alignment for tables, these have been removed in favour of system module's align classes which provided identical behaviour.

The classes were also used as configuration values in exported views, and an update has been provided to convert the views to the new class names. Distributions and modules with exported views configuration may need to be updated to reflect the new options.

---

## 42. [Url::createFromRequest does not ignore query parameters anymore](https://www.drupal.org/node/3522346)

- **Node ID**: [3522346](https://www.drupal.org/node/3522346)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2985400](https://www.drupal.org/node/2985400)

**Description:**

When creating a URL object via \Drupal\Core\Url::createFromRequest(), you do not need to set the missing query parameters via setOption anymore.
**Before:**

```php
$request = \Symfony\Component\HttpFoundation\Request::create('/node', 'GET');
    $url = \Drupal\Core\Url::createFromRequest($request);
    // Hack.
    $url->setOption('query', $request->query->all());
```

**After:**

```php
$request = \Symfony\Component\HttpFoundation\Request::create('/node', 'GET');
    $url = \Drupal\Core\Url::createFromRequest($request);
    // No hack anymore \o/.
```

---

## 43. [All transactions in core are converted to use explicit ::commitOrRelease(), implicit commit-on-destruct is deprecated](https://www.drupal.org/node/3524461)

- **Node ID**: [3524461](https://www.drupal.org/node/3524461)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3495728](https://www.drupal.org/node/3495728), [#3406985](https://www.drupal.org/node/3406985), [#3583849](https://www.drupal.org/node/3583849), [#3584238](https://www.drupal.org/node/3584238)

**Description:**

Drupal by default was committing a database transaction when its Transaction object goes out of scope, during the Transaction object destruction.

The new Transaction::commitOrRelease() method is now being used all across Drupal's core codebase to **explicitly** perform a commit/savepoint release operation on a database transaction.

Besides:

- The implicit commit on Transaction destruction behaviour is deprecated, and its implementation will be removed in Drupal 13.

- The methods introduced in [#3405976: Transaction autocommit during shutdown relies on unreliable object destruction order (xdebug 3.3+ enabled)](/project/drupal/issues/3405976) are deprecated for removal in Drupal 13, without replacement:

Connection::commitAll()

- Database::commitAllOnShutdown()

- TransactionManagerBase::commitAll()

- The following methods in Drupal\pgsql\Driver\Database\pgsql\Connection are deprecated for removal in Drupal 13:

::addSavepoint() replaced by Connection::startTransaction()

- ::releaseSavepoint() replaced by Transaction::commitOrRelease()

- ::rollbackSavepoint() replaced by Transaction::rollback()

Read [Transaction::commitOrRelease() method introduced to explicit commit a transaction](https://www.drupal.org/node/3512006) for before/after examples.

---

## 44. [Block content attributes are moved to the content wrapper](https://www.drupal.org/node/3525287)

- **Node ID**: [3525287](https://www.drupal.org/node/3525287)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2486267](https://www.drupal.org/node/2486267)

**Description:**

The attributes of a block content render array, in a #attributes key, are merged into the content's wrapper.

To apply attributes to the entire block, set the attributes in the render array #wrapper_attributes key instead.

Previously, if a block's content render array contained an #attributes key, then those attributes would be merged into the &lt;div&gt; wrapper for the entire block.

**Example:**

Given a render array like:

```php
return [
  '#attributes' => [
    'class' => [
      'foo'
    ],
    'data-content-custom' => 'bar',
  ],
  '#wrapper_attributes' => [
    'data-wrapper-custom' => 'baz',
  ],
  '#markup' => 'Sample content',
];
```

**Before:**

```php


    Sample content.


```

**After:**

```php


    Sample content.


```

---

## 45. [Brotli compression support added for CSS and JavaScript aggregates](https://www.drupal.org/node/3526344)

- **Node ID**: [3526344](https://www.drupal.org/node/3526344)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3184242](https://www.drupal.org/node/3184242)

**Description:**

Drupal now supports Brotli compression for aggregated CSS and JavaScript files in addition to the existing gzip compression. Brotli typically provides 15-25% better compression ratios than gzip, resulting in smaller file sizes and faster page loads for browsers that support it.

**What changed:**

- The separate system.performance.css.gzip and system.performance.js.gzip settings have been replaced with unified system.performance.css.compress and system.performance.js.compress boolean settings.**- When compression is enabled, Drupal will automatically generate both .gz (gzip) and .br (Brotli) versions of aggregated CSS and JavaScript files if the server has the necessary PHP extensions installed.
- The .htaccess file has been updated to serve Brotli-compressed files when available, with gzip as a fallback.
- Brotli files are served with higher priority than gzip files for browsers that support both compression methods.

**Server requirements:**

To generate Brotli-compressed files, the server must have:

- The PHP Brotli extension installed (ext-brotli)
- For Apache: The mod_brotli module is not required as files are pre-compressed
- For Nginx: Manual configuration updates are required (see below)

**Impact:**

**Site administrators:**

- Sites with gzip compression enabled will have compression enabled for both gzip and Brotli.
- No manual intervention is required for Apache-based sites.

**System administrators  :**

- Apache users**: The updated .htaccess rules will handle Brotli files automatically.**- Nginx users**: You need to update your Nginx configuration to serve Brotli files. Add the following to your server block:
nginx - see [documentation](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/brotli/)

```php
location ~ ^/sites/.*/files/css/(.*)\.css$ {
    gzip_static on;
    brotli_static on;
    try_files $uri.br $uri.gz $uri =404;
}

location ~ ^/sites/.*/files/js/(.*)\.js$ {
    gzip_static on;
    brotli_static on;
    try_files $uri.br $uri.gz $uri =404;
}
```

Note: The brotli_static directive requires the ngx_brotli module to be installed.

**Module developers:**

- If your module programmatically sets the compression configuration, update your code:

**Before:**

```php
$config->set('css.gzip', TRUE);
$config->set('js.gzip', TRUE);
```

**After:**

```php
$config->set('css.compress', TRUE);
$config->set('js.compress', TRUE);
```

- The AssetDumper service now automatically creates both .gz and .br files when compression is enabled and the server supports it.

**API changes:**

- **Deprecated**: system.performance.css.gzip and system.performance.js.gzip configuration settings**- Added**: system.performance.css.compress and system.performance.js.compress configuration settings**- The AssetDumper::dump() method now generates multiple compressed versions based on available PHP extensions

**Upgrade path:**

An post-update hook (system_post_update_migrate_compress_setting()) automatically migrates existing gzip settings to the new compression settings. Sites with gzip enabled will have compression enabled; sites with gzip disabled will have compression disabled.

**Benefits:**

- Improved performance**: Brotli typically achieves 15-25% better compression than gzip**- Backward compatibility**: Gzip files continue to be generated for older browsers**- Automatic fallback**: Browsers that don't support Brotli will automatically receive gzip-compressed files**- Future-proof**: The new architecture makes it easier to add support for additional compression algorithms in the future

---

## 46. [New method getSummary() added to Drupal\Core\Field\FieldTypeCategoryInterface](https://www.drupal.org/node/3526434)

- **Node ID**: [3526434](https://www.drupal.org/node/3526434)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3370326](https://www.drupal.org/node/3370326)

**Description:**

The getSummary() method has been added to Drupal\Core\Field\FieldTypeCategoryInterface, and it is implemented in Drupal\Core\Field\FieldTypeCategory.

If a class implements FieldTypeCategoryInterface and does not extend FieldTypeCategory, then it will have to be updated to implement the new method.

When the site builder adds a new field from /admin/structure/types/manage/page/fields (for example) and chooses a field-type category, the text from getSummary() will be shown on the form. For example, the options module adds the summary text below the Label field in this screenshot:

A module that adds a field-type category can provide summary text by adding the summary key to the YAML file. For example, the summary text in the screenshot comes from core/modules/options/options.field_type_categories.yml:

```php
selection_list:
  label: 'Selection list'
  description: 'Create text or number based list with predefined options'
  summary: 'Each list item has a Name, with limited formatting, and a Value. Values that are in use cannot be changed, since they are stored in the database. The Names can be edited later.The field type examples use the format Name (Value): Name is displayed to the user, and Value is stored in the database.'
  weight: -15
  libraries:
    - options/drupal.options-icon
```

It is safe to add the summary key to mymodule.field_type_categories.yml: if mymodule is used with earlier versions of Drupal core, then the summary will be ignored.
Like the values of the existing label and description keys, the new value is translatable.

---

## 47. [New permission available to view unpublished block content](https://www.drupal.org/node/3533073)

- **Node ID**: [3533073](https://www.drupal.org/node/3533073)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3020938](https://www.drupal.org/node/3020938)

**Description:**

The new view unpublished block content permission is now available to grant users the ability to view unpublished block content entities.

Previously a user had to have either administer block content or access block library permissions.

---

## 48. [System menu blocks have configuration option for "Add a CSS class to ancestors of the current page"](https://www.drupal.org/node/3533514)

- **Node ID**: [3533514](https://www.drupal.org/node/3533514)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3529464](https://www.drupal.org/node/3529464)

**Description:**

System menu blocks now have a configuration option for " "Add a CSS class to ancestors of the current page". The configuration option exposes behaviour which was previously always on and unconfigurable.

When sites aren't using the menu active trail behaviour to style menus, which may be the case when the menu only has one level of items, or when all items are expanded, this option can be disabled to considerably improve cache hit rates for menu rendering.

The behavior for existing menu blocks will remain the same unless explicitly changed in the block configuration.

---

## 49. [node_access_rebuild functions are deprecated](https://www.drupal.org/node/3534610)

- **Node ID**: [3534610](https://www.drupal.org/node/3534610)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3533299](https://www.drupal.org/node/3533299)

**Description:**

The procedural functions node_access_rebuild() and node_access_needs_rebuild()  have been deprecated.

Use \Drupal::service('Drupal\node\NodeAccessRebuild')-&gt;needsRebuild() and \Drupal::service('Drupal\node\NodeAccessRebuild')-&gt;setNeedsRebuild() instead of node_access_needs_rebuild()

Use \Drupal::service('Drupal\node\NodeAccessRebuild')-&gt;rebuild() instead of node_access_rebuild()

The internal functions _node_access_rebuild_batch_operation() and_node_access_rebuild_batch_finished() have also been removed and replaced with static methods in the new service class NodeAccessRebuild.

---

## 50. [HTML5 validation will be disabled in Drupal 12](https://www.drupal.org/node/3537128)

- **Node ID**: [3537128](https://www.drupal.org/node/3537128)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#1797438](https://www.drupal.org/node/1797438)

**Description:**

A new setting for controlling HTML5 validation, enable_html5_validation,  is added to the settings.php file.

This allows sites to decide if the wish to use client-side HTML validation or not. Using client-side HTML5 validation (e.g., required, pattern), can cause accessibility and user experience issues. On form submit with missing or invalid input, the browser will display its own validation messages and prevent the Drupal's Form API (FAPI) validation from executing. Thus, Drupal error messages are not displayed.

The results in a poor user experience, especially for those relying on screen readers, keyboard navigation, or translated interfaces. Native browser error messages cannot be customized, styled, or localized, and only appear for one field at a time.

In Drupal 11 this setting defaults to TRUE and HTML5 validation is enabled which retains the existing behavior.

In Drupal 12 this setting defaults to FALSE and HTML5 validation is disabled.

In Drupal 13 this setting is scheduled to be removed.

**To change the default behavior:**

**Drupal 11:**

Sites that want to disable HTML5 validation can do so by setting 'enable_html5_validation' to FALSE in settings.php.

$settings['enable_html5_validation'] = FALSE;

**Drupal 12:**

Sites that want to enable HTML5 validation can do so by setting 'enable_html5_validation' to TRUE in settings.php.

$settings['enable_html5_validation'] = TRUE;

**Note:** Drupal has a feature for making a form field required using JavaScript. For these fields with HMTL5 validation disabled, it is possible to submit a form even when they are empty. For more information, see [#3592742: States API required fields don't work without HTML5 validation](/project/drupal/issues/3592742).

---

## 51. [Disabled links are now ignored in active trail](https://www.drupal.org/node/3544512)

- **Node ID**: [3544512](https://www.drupal.org/node/3544512)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2973515](https://www.drupal.org/node/2973515)

**Description:**

Before this change, Drupal did not check if the matching menu link was enabled or disabled when retrieving the active menu trail.
This could lead to several problems when a menu link is disabled, such as an "active" class inappropriately set on a parent menu link, or issues with contrib modules like Menu Breadcrumb.
Disabled menu links are now ignored when building the active trail.

---

## 52. [expectDeprecation() is deprecated](https://www.drupal.org/node/3545276)

- **Node ID**: [3545276](https://www.drupal.org/node/3545276)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3497124](https://www.drupal.org/node/3497124), [#3552827](https://www.drupal.org/node/3552827)

**Description:**

Deprecation expectations in tests were introduced through symfony/phpunit-bridge well before PHPUnit started providing native support.

This was happening by marking deprecated scope with the @group legacy annotation and by expecting deprecations via calls to expectDeprecation().

When we started using PHPUnit 10, Drupal dropped the dependency to symfony/phpunit-bridge, and added our own custom DeprecationHandler to replace it, retaining the same logic.

In the meantime, PHPUnit evolved and in version 11.0, the expectUserDeprecationMessage() and expectUserDeprecationMessageMatches() methods were introduced with basically the same purpose.

Usage of @group legacy has already been replaced by the #[IgnoreDeprecations] attribute as part of [#3535662: [meta] Convert test metadata from annotations to attributes](/project/drupal/issues/3535662).

expectDeprecation() is now deprecated in favor of the expectUserDeprecationMessage() and expectUserDeprecationMessageMatches() methods.

Besides, the entire ExpectDeprecationTrait is deprecated. Normally, this trait would not have to be imported in concrete tests as it is already being imported in the base test classes (Unit, Kernel, Functional, Build) from where concrete tests extend from.
**Differences in PHPUnit implementation vs. Symfony legacy:**
Please note the following differences in behavior of the native PHPUnit implementation.

- When using expectUserDeprecationMessage(), the deprecation message must be exact, i.e. you can no longer use Symfony-style %x placeholders to match patterns. The expectUserDeprecationMessageMatches() method can be used to match patterns using regular expressions instead, in case that's needed.

- Multiple calls to expectDeprecation() in legacy code were also checking that the collected deprecations were matching a sequence. The new methods do not check that, only whether at least one of the collected deprecations matches the expected one, on a one-by-one basis.

- Deprecations can NO LONGER be triggered in ::tearDown() or methods tagged with the #[After] attribute - due to internal PHPUnit logic that stops collecting deprecations during test finalization.

- Use of expectUserDeprecationMessage() and expectUserDeprecationMessageMatches() methods is NO LONGER restricted on deprecation tests (i.e. tests marked with #[IgnoreDeprecations]) only. You can expect deprecations in any test and it's PHPUnit configuration deciding whether the test should also fail or not because of the deprecation.

**Before:**

```php
/**
- Tests the deprecation notices of the block theme.
- * @group legacy
 */
class BlockThemeDeprecationTest extends UnitTestCase {

  /**
- Tests the deprecation in the constructor.
   */
  public function testConstructorDeprecation(): void {
    $this->expectDeprecation('Calling Drupal\block\Plugin\migrate\process\BlockTheme::__construct() with the $migration argument is deprecated in drupal:10.1.0 and is removed in drupal:11.0.0. See https://www.drupal.org/node/3323212');
    ...
  }

}
```

**After:**

```php
...
use PHPUnit\Framework\Attributes\IgnoreDeprecations;
...

/**
- Tests the deprecation notices of the block theme.
 */
#[IgnoreDeprecations]
class BlockThemeDeprecationTest extends UnitTestCase {

  /**
- Tests the deprecation in the constructor.
   */
  public function testConstructorDeprecation(): void {
    $this->expectUserDeprecationMessage('Calling Drupal\block\Plugin\migrate\process\BlockTheme::__construct() with the $migration argument is deprecated in drupal:10.1.0 and is removed in drupal:11.0.0. See https://www.drupal.org/node/3323212');
    ...
  }

}
```

---

## 53. [CommentInterface::ANONYMOUS_* constants are deprecated](https://www.drupal.org/node/3547352)

- **Node ID**: [3547352](https://www.drupal.org/node/3547352)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3547349](https://www.drupal.org/node/3547349)

**Description:**

Constants for anonymous contact settings are deprecated. Use \Drupal\comment\AnonymousContact enum instead.

Before:
CommentItemInterface::ANONYMOUS_MAYNOT_CONTACT
CommentItemInterface::ANONYMOUS_MAY_CONTACT
CommentItemInterface::ANONYMOUS_MUST_CONTACT
After:
\Drupal\comment\AnonymousContact::Forbidden
\Drupal\comment\AnonymousContact::Allowed
\Drupal\comment\AnonymousContact::Required

---

## 54. [CommentItemInterface constants HIDDEN, OPEN and CLOSED are deprecated](https://www.drupal.org/node/3547362)

- **Node ID**: [3547362](https://www.drupal.org/node/3547362)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3547353](https://www.drupal.org/node/3547353)

**Description:**

CommentItemInterface constants HIDDEN, OPEN and CLOSED are deprecated. Use enum \Drupal\comment\CommentingStatus instead.

Before:

```php
CommentItemInterface::HIDDEN
CommentItemInterface::OPEN
CommentItemInterface::CLOSED
```

After:

```php
\Drupal\comment\CommentingStatus::Hidden
\Drupal\comment\CommentingStatus::Open
\Drupal\comment\CommentingStatus::Closed
```

---

## 55. [ToStringTrait is deprecated](https://www.drupal.org/node/3548961)

- **Node ID**: [3548961](https://www.drupal.org/node/3548961)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3548957](https://www.drupal.org/node/3548957)

**Description:**

The trait is deprecated because since PHP 7.4 exception handling is not required and since PHP 8.0 there's [Stringable](https://www.php.net/manual/class.stringable.php) interface

__toString method must be implemented directly without using \Drupal\Component\Utility\ToStringTrait.

**Before:**

```php
use \Drupal\Component\Utility\ToStringTrait;

class Example {

  use ToStringTrait;

  public function render(): string {
    return 'Example';
  }

}
```

**After:**

```php
class Example implements \Stringable {

  public function render(): string {
    return 'Example';
  }

  /**
- {@inheritdoc}
   */
  public function __toString(): string {
    return $this->render();
  }

}
```

---

## 56. [Using a #access value other than a boolean or an AccessResultInterface object is deprecated](https://www.drupal.org/node/3549344)

- **Node ID**: [3549344](https://www.drupal.org/node/3549344)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3526250](https://www.drupal.org/node/3526250)

**Description:**

The  '#access' key in render array elements must now be either an AccessResultInterface object or FALSE. The use any other type or value is deprecated.

**Before:**

Any #access value that is not strictly FALSE or an AccessResultInterface object is set to TRUE.

```php
$element = [
  '#markup' => 'foo',
  '#access' => new \stdClass(),
];
```

Results in a deprecation message.

**After:**

Modules should make sure that they use a valid value in #access. For example,

```php
$element = [
  '#markup' => 'foo',
  '#access' => \Drupal\Core\Access\AccessResultInterface::isAllowed,
];
```

---

## 57. [CommentItemInterface constants FORM_SEPARATE_PAGE and FORM_BELOW are deprecated](https://www.drupal.org/node/3550055)

- **Node ID**: [3550055](https://www.drupal.org/node/3550055)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3550054](https://www.drupal.org/node/3550054)

**Description:**

Constants for comment form display settings are deprecated. Use \Drupal\comment\FormLocation enum instead.

Before:

```php
CommentItemInterface::FORM_SEPARATE_PAGE
CommentItemInterface::FORM_BELOW
```

After:

```php
\Drupal\comment\FormLocation::SeparatePage
\Drupal\comment\FormLocation::Below
```

---

## 58. [Cursor offset and orientation arguments in StatementInterface::fetch() are deprecated](https://www.drupal.org/node/3551924)

- **Node ID**: [3551924](https://www.drupal.org/node/3551924)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3522561](https://www.drupal.org/node/3522561)

**Description:**

Cursor offset and orientation arguments in StatementInterface::fetch() are untested and likely not working.

Their usage is now triggering a deprecation, and the arguments will be removed in Drupal 12.

Before:

```php
$connection->query("SELECT * FROM {node}")->fetch(FetchAs::Associative, 1, 1);
```

After:

```php
$connection->query("SELECT * FROM {node}")->fetch(FetchAs::Associative);
```

---

## 59. [user.pass.http, user.login.http, user.login_status.http and user.logout.http routes moved to the rest module](https://www.drupal.org/node/3552724)

- **Node ID**: [3552724](https://www.drupal.org/node/3552724)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3530640](https://www.drupal.org/node/3530640)

**Description:**

There routes are now deprecated:

- user.pass.http (/user/password)

- user.login.http (/user/login)

- user.login_status.http (/user/login_status)

- user.logout.http (/user/logout)

(These are POST routes handling JSON requests, not the GET routes used by most users.)

They are being replaced by the following routes provided by the rest module:

- rest.pass (/user/password)

- rest.login (/user/login)

- rest.login_status (/user/login_status)

- rest.logout (/user/logout)

If you rely on these routes, you will need to enable the rest module.

If you have code that interacts with these route, you need to update it to use the new routes.

Before:

```php
$url = Url::fromRoute('user.login.http');
```

After:

```php
$url = Url::fromRoute('rest.login');
```

If you code calls or extends \Drupal\user\Controller\UserAuthenticationController, you should use \Drupal\rest\Controller\RestAuthenticationController instead.

---

## 60. [The function _update_cron_notify() has been removed](https://www.drupal.org/node/3552870)

- **Node ID**: [3552870](https://www.drupal.org/node/3552870)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3125013](https://www.drupal.org/node/3125013)

**Description:**

The function _update_cron_notify()is removed. There is no replacement.

---

## 61. [Drupal now uses symfony/runtime for bootstrap separation](https://www.drupal.org/node/3553275)

- **Node ID**: [3553275](https://www.drupal.org/node/3553275)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3313404](https://www.drupal.org/node/3313404)

**Description:**

Drupal now integrates the [Symfony Runtime component](https://symfony.com/doc/7.3/components/runtime.html) to separate bootstrapping logic from request handling. The change introduces a new DrupalRuntime class and modifies front-controllers (index.php, update.php) to use the runtime pattern instead of directly instantiating and handling requests through the DrupalKernel.

This issue does not remove bootstrapping logic from the DrupalKernel but enables this to be done in follow-up issues.

The Symfony Runtime component,  symfony/runtime, is added a dependency.

If you maintain custom front-controllers, you have two options:

- **Keep existing format**: Legacy front-controllers continue to work unchanged for the time being.

- **Adopt Symfony Runtime**: Update to the new pattern for better performance and features.

Front-controller example**

Before

```php
use Drupal\Core\DrupalKernel;
use Symfony\Component\HttpFoundation\Request;

$autoloader = require_once 'autoload.php';
$kernel = new DrupalKernel('prod', $autoloader);
$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

After

```php
use Drupal\Core\DrupalKernel;
use Drupal\Core\Runtime\DrupalRuntime;

$_ENV['APP_RUNTIME'] ??= $_SERVER['APP_RUNTIME'] ?? DrupalRuntime::class;
require_once 'autoload_runtime.php';

return static function () {
  return new DrupalKernel('prod', require 'autoload.php');
};
```

Note:**
It is important your script changes the current directory to Drupal's web root directory, so if was doing so before, it will need to after.

---

## 62. [LinkWidget::validateTitleElement() is deprecated](https://www.drupal.org/node/3554139)

- **Node ID**: [3554139](https://www.drupal.org/node/3554139)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3093118](https://www.drupal.org/node/3093118)

**Description:**

LinkWidget::validateTitleElement() is deprecated.  Validation was moved to the LinkTitleRequiredConstraintValidator on the LinkItem field plugin instead.  If your custom widget is for field_types: ['link'], then you can remove validateTitleElement() from your element's form validation with no replacement.

Before:

```php
public function formElement(FieldItemListInterface $items, $delta, array $element, array &$form, FormStateInterface $form_state) {
  $element['#element_validate'][] = [static::class, 'validateTitleElement'];
  return $element;
}
```

After:

```php
public function formElement(FieldItemListInterface $items, $delta, array $element, array &$form, FormStateInterface $form_state) {
  // No replacement.
  return $element;
}
```

---

## 63. [Using #item_attributes with image_formatter and responsive_image_formatter is deprecated](https://www.drupal.org/node/3554585)

- **Node ID**: [3554585](https://www.drupal.org/node/3554585)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3554447](https://www.drupal.org/node/3554447)

**Description:**

A few theme hooks used an item_attributes property instead of using the standard attributes.

To avoid special case handling, those theme hooks now use attributes.

**Before:**

```php
$elements[$delta] = [
        '#theme' => 'image_formatter',
        '#item_attributes' => [
          'loading' => $this->getSetting('image_loading')['attribute'],
        ],
        ...
      ];

      $elements[$delta] = [
        '#theme' => 'responsive_image_formatter',
        '#item_attributes' => $item_attributes,
        ...
      ];
```

**After:**

```php
$elements[$delta] = [
        '#theme' => 'image_formatter',
        '#attributes' => [
          'loading' => $this->getSetting('image_loading')['attribute'],
        ],
        ...
      ];

      $elements[$delta] = [
        '#theme' => 'responsive_image_formatter',
        '#attributes' => $item_attributes,  // Variable $item_attributes not renamed for easier comparison.
        ...
      ];
```

---

## 64. [Constraint plugins must use named arguments instead of an options array](https://www.drupal.org/node/3554746)

- **Node ID**: [3554746](https://www.drupal.org/node/3554746)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3555134](https://www.drupal.org/node/3555134), [#3555534](https://www.drupal.org/node/3555534), [#3561135](https://www.drupal.org/node/3561135)

**Description:**

Drupal's constraint plugins are a wrapper around the Symfony Validator component. Previously, constraint plugins were created by passing a single array of options to the constructor, e.g.

```php
$constraint = new MyConstraint(['message' => 'Custom validation message']);
```

In order to improve the API for both type safety and required arguments, Symfony has deprecated the single array format in favor of using named arguments:

```php
$constraint = new MyConstraint(message:  'Custom validation message');
```

Previously there was no need to write a constructor in most constraint plugins as the base Constraint constructor would take care of handling the options array. This feature will be removed in Symfony 8, which will ship alongside Drupal 12, so custom constraint plugins should add a custom constructor with named arguments.
If you do not need to provide backward compatibility you can use named arguments and constructor property promotion immediately:

```php
use Symfony\Component\Validator\Attribute\HasNamedArguments;
use Symfony\Component\Validator\Constraint;

class MyConstraint extends Constraint {

  #[HasNamedArguments]
  public function __construct(
    public string $message = 'Default validation message.',
    mixed ...$args,
  ) {
    parent::__construct(...$args);
  }

}
```

In order to ease the transition in contrib modules, you can provide backward compatibility as follows:

```php
use Symfony\Component\Validator\Attribute\HasNamedArguments;
use Symfony\Component\Validator\Constraint;

class MyConstraint extends Constraint {

  public string $message = 'Default validation message.';

  #[HasNamedArguments]
  public function __construct(?array $options = null, ?string $message = null, mixed ...$args) {
    if (is_array($options)) {
      @trigger_error(sprintf('Passing an array of options to configure the "%s" constraint is deprecated in drupal:11.3.0 and is removed in drupal:12.0.0. Use named arguments instead. See https://www.drupal.org/node/XXX', static::class), E_USER_DEPRECATED);
    }

    parent::__construct($options, ...$args);

    $this->message = $message ?? $this->message;
  }

}
```

The #[HasNamedArguments] attribute tells Symfony and Drupal that named arguments are supported when constructing the constraint.
In this example $message is the only configurable setting for the constraint but this should be extended to all settings that the constraint supports. The argument should be typed, and only nullable if the setting is optional.

In addition, passing any value that is not an associative array or NULL to the addConstraint() method of any class implementing the following interfaces is deprecated:

- Drupal\Component\Plugin\Context\ContextDefinitionInterface

- Drupal\Core\Entity\EntityTypeInterface

- Drupal\Core\Field\FieldConfigInterface

- Drupal\Core\TypedData\DataDefinitionInterface

In Drupal 12, the signature of those methods will look like this:**public function addConstraint(string $constraint_name, ?array $options = NULL): static;
The keys of the $options associative array are the names of the constraint properties being set, and the array values are the values to set for the corresponding properties.

For example:
Before**

```php
$properties['entity'] = DataReferenceDefinition::create('entity')->addConstraint('EntityType', 'taxonomy_term');
```

**Now:**

```php
$properties['entity'] = DataReferenceDefinition::create('entity')->addConstraint('EntityType', ['type' => 'taxonomy_term']);
```

Lastly, passing any value that is not an associative array or NULL to the craete() of Drupal\Core\Validation\ConstraintManager is deprecated. In Drupal 12, the signature will be updated to:
`public function create(string $name, ?array $options): \Symfony\Component\Validator\Constraint

---

## 65. [user_load_by_mail() and user_load_by_name() are deprecated](https://www.drupal.org/node/3555936)

- **Node ID**: [3555936](https://www.drupal.org/node/3555936)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3555670](https://www.drupal.org/node/3555670)

**Description:**

user_load_by_mail()  and user_load_by_name() are deprecated and will be removed in Drupal 13.  Load User entities from storage by their properties instead.

Before:

```php
$user = user_load_by_mail($mail);
```

After:

```php
$users = \Drupal::entityTypeManager()->getStorage('user')->loadByProperties(['mail' => $mail]);
$user = reset($users);
```

Before:

```php
$user = user_load_by_name($name);
```

After:

```php
$users = \Drupal::entityTypeManager()->getStorage('user')->loadByProperties(['name' => $name]);
$user = reset($users);
```

---

## 66. [Deprecated email addresses will no longer pass validation](https://www.drupal.org/node/3557004)

- **Node ID**: [3557004](https://www.drupal.org/node/3557004)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3521184](https://www.drupal.org/node/3521184)

**Description:**

Previously if an email address with a deprecated format was validated by a form the address would pass validation.  But mail transfer will usually fail with a warning because servers do not support it.  This included addresses with a space character between the local part (username) and '@' symbol, for example "john.doe @example.com".

The EmailValidator class has been updated to cause deprecated format addresses to fail validation instead.

---

## 67. [Introduces Database Schema Definition instead of array-based structure](https://www.drupal.org/node/3557405)

- **Node ID**: [3557405](https://www.drupal.org/node/3557405)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3411490](https://www.drupal.org/node/3411490), [#3557481](https://www.drupal.org/node/3557481), [#3558426](https://www.drupal.org/node/3558426)

**Description:**

This introduces the Database Schema Definition to replace the current way of defining a table's structure via a nested array, with a structure of value objects defined in the Drupal\Core\Database\SchemaDefinition namespace.

This structure is immutable and database abstract.

In the future, the db-specific field types in the existing array-based schema definition will no longer be defined here and database drivers will be able to implement db-specific features such as GIN and GIST indexes. It will also allow for better introspection of database schema, and validation of properties.

**Schema definition changes:**

**Before:**

```php
$schema = [
      'description' => 'The base table for configuration data.',
      'fields' => [
        'collection' => [
          'description' => 'Primary Key: Config object collection.',
          'type' => 'varchar_ascii',
          'length' => 255,
          'not null' => TRUE,
          'default' => '',
        ],
        'name' => [
          'description' => 'Primary Key: Config object name.',
          'type' => 'varchar_ascii',
          'length' => 255,
          'not null' => TRUE,
          'default' => '',
        ],
        'data' => [
          'description' => 'A serialized configuration object data.',
          'type' => 'blob',
          'not null' => FALSE,
          'size' => 'big',
        ],
      ],
      'primary key' => ['collection', 'name'],
    ];
```

**After:**

Note the usage of named parameters. In the future, these could allow more flexibility when it comes to adding, changing, deprecating, and removing function arguments.

```php
new Table(
      name: 'config',
      description: 'The base table for configuration data.',
      columns: [
        new Column(
          name: 'collection',
          description: 'Primary Key: Config object collection.',
          type: ColumnType::VarcharAscii,
          length: 255,
          notNull: TRUE,
          default: new StringValue(''),
        ),
        new Column(
          name: 'name',
          description: 'Primary Key: Config object name.',
          type: ColumnType::VarcharAscii,
          length: 255,
          notNull: TRUE,
          default: new StringValue(''),
        ),
        new Column(
          name: 'data',
          description: 'A serialized configuration object data.',
          type: ColumnType::Blob,
          notNull: FALSE,
          size: ColumnSize::Big,
        ),
      ],
      primaryKey: new PrimaryKey(['collection', 'name']),
    );
```

**hook_schema() changes:**

Refer to database.api.php for a full example.

**Before:**

```php
/**
- Implements hook_schema().
 */
function system_schema(): array {
  $schema['my_table'] = [
    ...
  ];

  return $schema;
}
```

**After:**

```php
/**
- Implements hook_schema().
 */
function system_schema(): Schema {
    $tables[] = new Table(
      name: 'my_table',
      ...
    );

    return new Schema(
      type: SchemaDefinitionType::Module,
      name: 'system',
      tables: $tables,
    );
}
```

**Storage handlers:**

**Before:**

```php
/**
- Check if the table exists and create it if not.
   */
  protected function ensureTableExists() {
    try {
      $database_schema = $this->connection->schema();
      $schema_definition = $this->schemaDefinition();
      $database_schema->createTable(static::TABLE_NAME, $schema_definition);
    }
    // If another process has already created the batch table, attempting to
    // recreate it will throw an exception. In this case just catch the
    // exception and do nothing.
    catch (DatabaseException) {
    }
    catch (\Exception) {
      return FALSE;
    }
    return TRUE;
  }

  public function schemaDefinition() {
    return [
      ...
    ];
  }
```

**After:**

```php
/**
- Check if the table exists and create it if not.
   */
  protected function ensureTableExists() {
    try {
      $this->connection->schema()->createSchemaFromDefinition($this->schemaDefinition());
    }
    // If another process has already created the batch table, attempting to
    // recreate it will throw an exception. In this case just catch the
    // exception and do nothing.
    catch (DatabaseException) {
    }
    catch (\Exception) {
      return FALSE;
    }
    return TRUE;
  }

  public function schemaDefinition(): Schema {
    $tables[] = new Table(
      name: 'my_storage_table',
      ...
    );

    return new Schema(
      type: SchemaDefinitionType::Storage,
      name: 'my_storage',
      tables: $tables,
    );
  }
```

---

## 68. [New asset garbage collection threshold](https://www.drupal.org/node/3557835)

- **Node ID**: [3557835](https://www.drupal.org/node/3557835)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3505776](https://www.drupal.org/node/3505776)

**Description:**

A new $settings['asset_gc_threshold'] has been introduced, defaulting to 45 days. When caches are cleared, asset aggregates older than this threshold will be deleted, newer files will be retained. This replaces the previous behaviour where all asset aggregates would be deleted on each cache clear.

This change means that aggregates will not have to be recreated immediately after a cache clear, but only if the asset versions, or contents of unversioned assets, have changed in the code base, leading to much higher CDN and browser cache hit rates (thanks to more stable Etag and last-modified headers), and a reduction in the number of times the same file has to be regenerated by Drupal.

During development, especially of assets with a specified version, developers may want to set this threshold to 0 so that all files are deleted. Site owners should be able to leave this setting to the default 45 days.

---

## 69. [Drupal\text\Plugin\Field\FieldType\TextItemBase implements onDependencyRemoval()](https://www.drupal.org/node/3558008)

- **Node ID**: [3558008](https://www.drupal.org/node/3558008)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3537624](https://www.drupal.org/node/3537624)

**Description:**

Drupal\text\Plugin\Field\FieldType\TextItemBase implements onDependencyRemoval() with this function signature:

```php
public static function onDependencyRemoval(FieldDefinitionInterface $field_definition, array $dependencies): bool {
```

If a contrib or custom module extends TextItemBase and overrides this method, then it should use a compatible signature. In particular, it should include the return-type declaration. (The signature in the parent class, Drupal\Core\Field\FieldItemBase, does not include the return-type declaration.)

---

## 70. [Passing null as $deserialization_target_class to ResourceType is deprecated](https://www.drupal.org/node/3558394)

- **Node ID**: [3558394](https://www.drupal.org/node/3558394)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3557053](https://www.drupal.org/node/3557053)

**Description:**

The $deserialization_target_class constructor parameter of the \Drupal\jsonapi\ResourceType\ResourceType class does not have a native typehint. This will be added in Drupal 12. Passing NULL is deprecated. If the target class is not required, pass \stdClass::class.

---

## 71. [page_top and page_bottom can now be added using attachments on a page's main content](https://www.drupal.org/node/3558616)

- **Node ID**: [3558616](https://www.drupal.org/node/3558616)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3339905](https://www.drupal.org/node/3339905)

**Description:**

Before this change, hook_page_top and hook_page_bottom were the only way to inject content into the top and bottom of a page. Now it is possible to add this content directly from a Controller using the #attached key.

**Before:**

```php
/**
- Implements hook_page_top().
   */
  #[Hook('page_top')]
  public function pageTop(array &$page_top): void {
    if ($this->routeMatch->getRouteName() == 'my_route') {
      $page_top['my_content'] = [
        '#markup' => 'Hello world

,
      ];
    }
  }
```

**After:**

```php
// In the controller for my_route:
$build['#attached']['page_top']['my_module'] = [
  '#markup' => 'Hello world

,
];
```

---

## 72. [Drupal\Core\Render\MainContent\HtmlRenderer::buildPageTopAndBottom now has $page_top and $page_bottom parameters](https://www.drupal.org/node/3558617)

- **Node ID**: [3558617](https://www.drupal.org/node/3558617)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3339905](https://www.drupal.org/node/3339905)

**Description:**

Drupal\Core\Render\MainContent\HtmlRenderer::buildPageTopAndBottom now has $page_top and $page_bottom parameters. This is used to add to the page_top and page_bottom variables of a page.

See [https://www.drupal.org/node/3558616](https://www.drupal.org/node/3558616)

---

## 73. [navigation__message theme hook deleted.](https://www.drupal.org/node/3559492)

- **Node ID**: [3559492](https://www.drupal.org/node/3559492)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3502993](https://www.drupal.org/node/3502993)

**Description:**

Previously, the Umami profile added messages to the navigation module via hook navigation__message .
But this is no longer necessary because Navigation Message is now an SDC component.
The component is core/modules/navigation/components/message/message.component.yml.

---

## 74. [Symfony TypeInfo component added as a composer dependency](https://www.drupal.org/node/3559767)

- **Node ID**: [3559767](https://www.drupal.org/node/3559767)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3548968](https://www.drupal.org/node/3548968)

**Description:**

The Symfony [TypeInfo](https://symfony.com/doc/current/components/type_info.html) component has been added as a composer dependency. The Drupal ArgumentsResolver utility class now relies on TypeInfo to match arguments against method parameter type hints.

With this change, union and intersection types are supported in method signatures where ArgumentsResolver is used (access checks and rest resources).

---

## 75. [UUIDs are now validated](https://www.drupal.org/node/3560109)

- **Node ID**: [3560109](https://www.drupal.org/node/3560109)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3559874](https://www.drupal.org/node/3559874)

**Description:**

Core's uuid field type -- which is automatically used to store content entity UUIDs -- now validates that the value it holds is, in fact, a valid UUID.

Previously, this was not the case; only the value's length was validated. So as long as it fit within a certain size, you could have an invalid UUID like this-is-my-entity. Now, such a value will no longer pass validation.

Most people are unlikely to run into this, but it may cause any entities with invalid UUIDs to fail validation. If that happens to you, you must change the entity's UUID to something valid:

```php
$entity->set('uuid', \Drupal::service('uuid')->generate())->save();
```

---

## 76. [dblog_filters and _dblog_get_message_types have been deprecated.](https://www.drupal.org/node/3560399)

- **Node ID**: [3560399](https://www.drupal.org/node/3560399)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3560398](https://www.drupal.org/node/3560398)

**Description:**

dblog_filters and_dblog_get_message_types have been deprecated.

Use \Drupal::service(\Drupal\dblog\DbLogFilters::class)-&gt;filters() or \Drupal::service(\Drupal\dblog\DbLogFilters::class)-&gt;getMessageTypes() as needed.

The file core/modules/dblog/dblog.admin.inc has also been deprecated.

---

## 77. [New MenuLinkTreeManipulatorsAlterEvent event added](https://www.drupal.org/node/3560671)

- **Node ID**: [3560671](https://www.drupal.org/node/3560671)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3091246](https://www.drupal.org/node/3091246)

**Description:**

A new MenuLinkTreeManipulatorsAlterEvent event has been added, which provides the ability to manipulate the menu link tree.

**Code example:**

```php
class MenuLinkTreeEventSubscriber implements EventSubscriberInterface {

  public static function alterMenuLinkManipulators(MenuLinkTreeManipulatorsAlterEvent $event): void {
    // Custom code.
  }

  public static function getSubscribedEvents(): array {
    return [
      MenuLinkTreeManipulatorsAlterEvent::class => ['alterMenuLinkManipulators'],
    ];
  }
}
```

---

## 78. [Single cardinality entity fields are now loaded from the database at once](https://www.drupal.org/node/3562172)

- **Node ID**: [3562172](https://www.drupal.org/node/3562172)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3551308](https://www.drupal.org/node/3551308), [#3564689](https://www.drupal.org/node/3564689)

**Description:**

Single and multiple cardinality entity fields are now loaded from the database in two database queries, instead of a query per field.

This can reduce the number of database queries on a page by tens or hundreds; depending on the number of fields being loaded, and the number of different entity load calls on the page.

Note that this change means that more data is loaded from the database at once. In extreme situations, either when loading many thousands of entities in one go, or if entities have very large blob fields, e.g. large amounts of text, this could cause queries that would previously be within MySQL's max_allowed_packet (which defaults to 32MB for the MySQL server) limit to exceed it.

Should this happen, it's recommended to avoid loading an unbounded or very high number of entities at once. Instead you should ensure that views or entity queries always have a limit. For bulk operations, entities should be loaded in chunks, e.g. see e.g. entity_update_batch_size. These approaches ensure that both PHP and MySQL memory usage remains within reasonable constraints. If you have control over MySQL configuration, it is also possible to increase max_allowed_packet temporarily before making code changes.

---

## 79. [ImageToolkit and ImageToolkitOperation plugins are autowirable](https://www.drupal.org/node/3562304)

- **Node ID**: [3562304](https://www.drupal.org/node/3562304)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3559481](https://www.drupal.org/node/3559481)

**Description:**

ImageToolkit and ImageToolkitOperation plugins are now autowirable.

---

## 80. [justinrainbow/json-schema moved to a production dependency of Drupal core](https://www.drupal.org/node/3563143)

- **Node ID**: [3563143](https://www.drupal.org/node/3563143)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3562627](https://www.drupal.org/node/3562627), [#3365985](https://www.drupal.org/node/3365985)

**Description:**

justinrainbow/json-schema has been a development dependency of core for some time. It has now been moved to a production dependency to support [#3352063: Allow schema references in Single Directory Component prop schemas](/project/drupal/issues/3352063).

---

## 81. [New KeyValueStoreInterface::getAllKeys() method](https://www.drupal.org/node/3563733)

- **Node ID**: [3563733](https://www.drupal.org/node/3563733)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3563639](https://www.drupal.org/node/3563639)

**Description:**

A method to get all keys in a KeyValue collection, getAllKeys(), has been added to \Drupal\Core\KeyValueStore\KeyValueStoreInterface.

This method lets you know the keys of the items in a given collection without the need of loading all those items. This can help to reduce memory usage where a collection has a large number of items or when the items themselves are large.

---

## 82. [Method getCacheTags() added to Drupal\Core\Field\LinkItemInterface](https://www.drupal.org/node/3564800)

- **Node ID**: [3564800](https://www.drupal.org/node/3564800)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3135042](https://www.drupal.org/node/3135042)

**Description:**

A new method, getCacheTags() has been added to Drupal\Core\Field\LinkItemInterface.  The original purpose of the method is to add entity cache metadata to rendered links.  This will prevent Drupal from displaying cached links with outdated paths, for instance in the event of a node's path alias being updated.

Method signature:

```php
/**
- Gets an array of cache tags for the link.
- * @return array
- The cache tags.
   */
  public function getCacheTags(): array;
```

An example of how this function is used can be found in Drupal\link\Plugin\Field\FieldFormatter\LinkFormatter::viewElements().

```php
/**
- {@inheritdoc}
   */
  public function viewElements(FieldItemListInterface $items, $langcode) {
    $element = [];
    $cache_tags = [];

    foreach ($items as $delta => $item) {
      // Complicated logic for building the render array.
      $cache_tags = Cache::mergeTags($cache_tags, $item->getCacheTags());
    }

    if ($cache_tags) {
      $element['#cache']['tags'] = $cache_tags;
    }
    return $element;
  }
```

---

## 83. [Views CachePluginBase::getRowCacheKeys() deprecated, row-level caching removed](https://www.drupal.org/node/3564958)

- **Node ID**: [3564958](https://www.drupal.org/node/3564958)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3564937](https://www.drupal.org/node/3564937)

**Description:**

- CachePluginBase::getRowCacheKeys() has been deprecated for removal in Drupal 13.0.0 without replacement.

- ::getRowCacheKeys() will not be called on views cache plugins from 11.4.0 onwards.

The individual rows of a view are usually cheap to render, and the row caching resulted in a large number of cache entries with low cache hit rates.

**Before:**

Views cached the result of individual result rows in the render cache.

**After:**

Views does not cache the result of individual result rows in the render cache. Views render output is still cached at the level of the display.

---

## 84. [UUID support for revisions](https://www.drupal.org/node/3565175)

- **Node ID**: [3565175](https://www.drupal.org/node/3565175)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#1812202](https://www.drupal.org/node/1812202)

**Description:**

TBD

---

## 85. [New views.grid and display link asset libraries](https://www.drupal.org/node/3565395)

- **Node ID**: [3565395](https://www.drupal.org/node/3565395)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3565297](https://www.drupal.org/node/3565297)

**Description:**

The CSS for the views grid style plugin has been moved to its own library, it was previously added on every page with a view regardless of style plugins used.

This change only applies to the legacy, non-responsive views plugin, the 'Responsive grid' views plugin is unaffected.

Additionally the CSS for the 'display link' area plugin has been moved to its own library.

Themes that are overriding these libraries with stylesheets_remove or stylesheets_replace will need to update the libraries they target.

---

## 86. [Upsert queries can now use unique / primary key constraints composed of multiple fields](https://www.drupal.org/node/3565717)

- **Node ID**: [3565717](https://www.drupal.org/node/3565717)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2547493](https://www.drupal.org/node/2547493)

**Description:**

Upsert database queries can now be used on tables with unique/primary keys composed of multiple fields.

For this purpose, a new \Drupal\Core\Database\Query\Upsert::keyColumns() method was added, and \Drupal\Core\Database\Query\Upsert::key() has been deprecated.

Usage example for a table with a composite primary key on the collection and name columns:

```php
$this->connection->upsert($this->table)
      ->key(['collection', 'name'])
      ->fields([
        'collection' => $collection,
        'name' => $key,
        'value' => $value,
      ])
      ->execute();
```

---

## 87. ['View linked label' operation added to user entity](https://www.drupal.org/node/3565758)

- **Node ID**: [3565758](https://www.drupal.org/node/3565758)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3506444](https://www.drupal.org/node/3506444)

**Description:**

The user entity access control handler now handles the 'view linked label' operation (see Drupal\user\UserAccessControlHandler::checkAccess()).

The intent of this entity operation is to determine whether a username theme element can be displayed as a link to the user profile. It differs from the 'view' operation in that it disregards whether a user is viewing their username, and in doing so, does not require the user cache context.

For example, the previous expected functionality was:

- All users with the 'access user profiles' permission see any node author name linked to the author's profile

- Any user viewing a node they authored sees the author name linked to their own profile

- All users not in the first two groups see the node author name as static text

Now, the expected functionality is:

- All users with the 'access user profiles' permission see any node author name linked to the author's profile

- All users without the 'access user profiles' permission see the node author name as static text

---

## 88. [Redundant WAI-ARIA `role` attributes removed from templates](https://www.drupal.org/node/3566225)

- **Node ID**: [3566225](https://www.drupal.org/node/3566225)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2655794](https://www.drupal.org/node/2655794)

**Description:**

WAI-ARIA role attributes that duplicated the native semantics of their elements have been removed. These attributes were originally added to support browsers that did not correctly infer semantic roles. Those browsers are no longer supported by Drupal core, making the additional ARIA roles unnecessary.

Elements updated

Before
After

&lt;aside role="complementary"&gt;
&lt;aside&gt;

&lt;footer role="contentinfo"&gt;
&lt;footer&gt;

&lt;header role="banner"&gt;
&lt;header&gt;

&lt;main role="main"&gt;
&lt;main&gt;

&lt;nav role="navigation"&gt;
&lt;nav&gt;

**Templates updated:**

- block--local-tasks-block.html.twig

- block--system-menu-block.html.twig

- breadcrumb.html.twig

- install-page.html.twig

- maintenance-page--front.html.twig

- maintenance-page.html.twig

- menu-local-tasks.html.twig

- page.html.twig

- pager.html.twig

- toolbar.html.twig

- views-mini-pager.html.twig

---

## 89. [The node/form library is deprecated](https://www.drupal.org/node/3566511)

- **Node ID**: [3566511](https://www.drupal.org/node/3566511)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2335523](https://www.drupal.org/node/2335523)

**Description:**

The node module has layout styling in node.module.css that is unused by core and contrib themes. This file was used by the node/form and node/drupal.node libraries

The node/drupal.node library no longer includes this file, and the node/form library has been deprecated.

The node.module.css file and the node/form library will be removed in Drupal 12.

---

## 90. [AutowireTrait supports setter injection with the #[Required] attribute](https://www.drupal.org/node/3566688)

- **Node ID**: [3566688](https://www.drupal.org/node/3566688)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3558306](https://www.drupal.org/node/3558306)

**Description:**

Classes using AutowireTrait or AutowiredInstanceTrait can now inject dependencies via setter methods marked with Symfony's #[Required] attribute, eliminating the need to override constructors in subclasses. This includes controllers that extend \Drupal\Core\Controller\ControllerBase and plugins that extend \Drupal\Core\Plugin\PluginBase.

For example, if there is a base plugin class like this:

```php
abstract class ExamplePluginBase extends PluginBase {

  public function __construct(
    protected readonly Service1Interface $service1,
    protected readonly Service2Interface $service2,
    protected readonly Service3Interface $service3,
  ) {}
  ...
}
```

a plugin class extending ExamplePluginBase would previously need to inject additional services by appending parameters to  the constructor:

```php
class ExamplePlugin1 extends ExamplePluginBase {

  public function __construct(
    Service1Interface $service1,
    Service2Interface $service2,
    Service3Interface $service3,
    protected readonly SubclassOnlyServiceInterface $subclassOnlyService,
  ) {
    parent::__construct($service1, $service2, $service3);
  }
  ...
}
```

Now the plugin can do this instead:

```php
use Symfony\Contracts\Service\Attribute\Required;

class ExamplePlugin1 extends ExamplePluginBase {

  protected SubclassOnlyServiceInterface $subclassOnlyService;

  #[Required]
  public function setSubclassOnlyService(SubclassOnlyServiceInterface $subclassOnlyService): void {
    $this->subclassOnlyService = $subclassOnlyService
  }
  ...
}
```

The trait will ensure that all #[Required] functions are called automatically when the controller or plugin is instantiated.
Services in the setter function are resolved by typehint or the #[Autowire] attribute, in the same way as the constructor.

---

## 91. [The drupal/core-dev-pinned metapackage is deprecated](https://www.drupal.org/node/3566742)

- **Node ID**: [3566742](https://www.drupal.org/node/3566742)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566600](https://www.drupal.org/node/3566600)

**Description:**

The meta-package drupal/core-dev-pinned is deprecated.  It will be removed in Drupal 12.

Instead, use the drupal/core-dev meta-package. This will allow dependency updates and avoid security vulnerabilities from pinned dependency versions.

---

## 92. [Several procedural submit, validation, Ajax callbacks and other functions were converted to methods and deprecated](https://www.drupal.org/node/3566774)

- **Node ID**: [3566774](https://www.drupal.org/node/3566774)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566768](https://www.drupal.org/node/3566768), [#3566792](https://www.drupal.org/node/3566792), [#3566888](https://www.drupal.org/node/3566888), [#3567163](https://www.drupal.org/node/3567163), [#3568092](https://www.drupal.org/node/3568092), [#3568124](https://www.drupal.org/node/3568124), [#3226806](https://www.drupal.org/node/3226806), [#3570235](https://www.drupal.org/node/3570235), [#3566536](https://www.drupal.org/node/3566536), [#3570238](https://www.drupal.org/node/3570238), [#3570863](https://www.drupal.org/node/3570863), [#3570839](https://www.drupal.org/node/3570839), [#3035340](https://www.drupal.org/node/3035340), [#3571400](https://www.drupal.org/node/3571400), [#3574727](https://www.drupal.org/node/3574727), [#3572173](https://www.drupal.org/node/3572173), [#3580662](https://www.drupal.org/node/3580662), [#941970](https://www.drupal.org/node/941970), [#3571172](https://www.drupal.org/node/3571172)

**Description:**

**Important note!:**

This CR is a living document, we have added the issues to the table so you can see if the issue is merged or not yet.

The following form submit, validation Ajax callbacks and functions were converted to methods and the procedural functions were deprecated:

Deprecated procedural callback
Converted to...
Progress

automated_cron_settings_submit()
\Drupal\automated_cron\Hook\AutomatedCronHooks::automatedCronSettingsSubmit()
[#3566768: Change \Drupal\system\Form\CronForm to use ConfigFormBase and use #config_target](/project/drupal/issues/3566768)

ckeditor5_filter_format_edit_form_submit()
\Drupal\ckeditor5\Hook\Ckeditor5Hooks::filterFormatEditFormSubmit()
[#3566792: Deprecate remaining ckeditor5.module procedural code](/project/drupal/issues/3566792)

_update_ckeditor5_html_filter()
\Drupal\ckeditor5\Hook\Ckeditor5Hooks::updateCkeditor5HtmlFilter()
[#3566792: Deprecate remaining ckeditor5.module procedural code](/project/drupal/issues/3566792)

contact_user_profile_form_submit()
\Drupal\contact\Hook\ContactFormHooks::profileFormSubmit()
[#3566888: Move contact module form callbacks from procedural to hook class](/project/drupal/issues/3566888)

contact_form_user_admin_settings_submit()
\Drupal\contact\Hook\ContactFormHooks::userAdminSettingsSubmit()
[#3566888: Move contact module form callbacks from procedural to hook class](/project/drupal/issues/3566888)

field_ui_form_manage_field_form_submit()
\Drupal\field_ui\Hook\FieldUiHooks::manageFieldFormSubmit()
[#3567163: Move field_ui form callback to hook class](/project/drupal/issues/3567163)

editor_form_filter_admin_format_editor_configure()
\Drupal\editor\Hook\EditorHooks::editorFormFilterAdminFormatEditorConfigure()
[#3568092: Convert editor.module procedural submit, validate and Ajax callbacks to methods](/project/drupal/issues/3568092)

editor_form_filter_admin_format_editor_configure()
\Drupal\editor\Hook\EditorHooks::editorFormFilterAdminFormatEditorConfigure()
[#3568092: Convert editor.module procedural submit, validate and Ajax callbacks to methods](/project/drupal/issues/3568092)

editor_form_filter_admin_format_validate()
\Drupal\editor\Hook\EditorHooks::editorFormFilterAdminFormatValidate()
[#3568092: Convert editor.module procedural submit, validate and Ajax callbacks to methods](/project/drupal/issues/3568092)

editor_form_filter_admin_format_submit()
\Drupal\editor\Hook\EditorHooks::editorFormFilterAdminFormatSubmit()
[#3568092: Convert editor.module procedural submit, validate and Ajax callbacks to methods](/project/drupal/issues/3568092)

language_process_language_select()
\Drupal\language\Hook\LanguageHooks::processLanguageSelect()
[#3574727: Deprecate remaining language.module code](/project/drupal/issues/3574727)

language_configuration_element_submit()
\Drupal\language\Element\LanguageConfiguration::submit()
[#3574727: Deprecate remaining language.module code](/project/drupal/issues/3574727)

media_filter_format_edit_form_validate()
\Drupal\media\Hook\MediaHooks::formatEditFormValidate()
[#3568124: Deprecate remaining functions in media.module](/project/drupal/issues/3568124)

_menu_ui_node_save()
\Drupal\menu_ui\Hook\MenuUiHooks:: menuUiNodeSave()
[#3571400: Deprecate functions in menu_ui.module and move to hooks or helper class](/project/drupal/issues/3571400)

menu_ui_get_menu_link_defaults()
\Drupal\menu_ui\Hook\MenuUiHooks:: getMenuLinkDefaults()
[#3571400: Deprecate functions in menu_ui.module and move to hooks or helper class](/project/drupal/issues/3571400)

menu_ui_node_builder()
\Drupal\menu_ui\Hook\MenuUiHooks:: nodeBuilder()
[#3571400: Deprecate functions in menu_ui.module and move to hooks or helper class](/project/drupal/issues/3571400)

menu_ui_form_node_form_submit()
\Drupal\menu_ui\Hook\MenuUiHooks:: formNodeFormSubmit()
[#3571400: Deprecate functions in menu_ui.module and move to hooks or helper class](/project/drupal/issues/3571400)

menu_ui_form_node_type_form_validate()
\Drupal\menu_ui\Hook\MenuUiHooks:: formNodeTypeFormValidate()
[#3571400: Deprecate functions in menu_ui.module and move to hooks or helper class](/project/drupal/issues/3571400)

menu_ui_form_node_type_form_builder()
\Drupal\menu_ui\Hook\MenuUiHooks:: formNodeTypeFormBuilder()
[#3571400: Deprecate functions in menu_ui.module and move to hooks or helper class](/project/drupal/issues/3571400)

- views_view_is_enabled

- views_view_is_disabled

- views_enable_view

- views_disable_view

\Drupal\views\ViewStatusTrait

- -&gt;isViewEnabled

- -&gt;isViewDisabled

- -&gt;enableView

- -&gt;disableView

[#3572243: Deprecate several views functions](/project/drupal/issues/3572243)

views_invalidate_cache
\Drupal\views\Views

- invalidateCache

- routeRebuildNeeded

[#941970: Only set router rebuild needed when something related to routing actually changes](/project/drupal/issues/941970)

views_ui_add_limited_validation()
\Drupal\views\ViewsFormAjaxHelperTrait::addLimitedValidation()
[#3035340: Deprecate core/modules/views_ui/admin.inc](/project/drupal/issues/3035340)

views_ui_add_ajax_wrapper()
\Drupal\views\ ViewsFormAjaxHelperTrait::addAjaxWrapper()
[#3035340: Deprecate core/modules/views_ui/admin.inc](/project/drupal/issues/3035340)

views_ui_ajax_update_form()
\Drupal\views\ ViewsFormAjaxHelperTrait::ajaxUpdateForm()
[#3035340: Deprecate core/modules/views_ui/admin.inc](/project/drupal/issues/3035340)

views_ui_nojs_submit()
\Drupal\views\ ViewsFormAjaxHelperTrait::noJsSubmit()
[#3035340: Deprecate core/modules/views_ui/admin.inc](/project/drupal/issues/3035340)

views_ui_form_button_was_clicked()
\Drupal\views\ViewsFormHelperTrait::formButtonWasClicked()
[#3035340: Deprecate core/modules/views_ui/admin.inc](/project/drupal/issues/3035340)

**The following functions have been removed or deprecated without replacement:**

Deprecated procedural callback
Progress

_ckeditor5_theme_css
[#3566792: Deprecate remaining ckeditor5.module procedural code](/project/drupal/issues/3566792)

- _field_create_entity_from_ids

- field_form_field_config_edit_form_entity_builder

[#3572173: Deprecate field.module remaining functions](/project/drupal/issues/3572173)

_filter_tips
[#3580662: Deprecate _filter_tips()](/project/drupal/issues/3580662)

- _filter_url

- _filter_url_parse_full_links

- _filter_url_parse_email_links

- _filter_url_parse_partial_links

- _filter_url_escape_comments

- _filter_url_trim

- _filter_autop

- _filter_html_escape

- _filter_html_image_secure_process

[#3226806: Move filter implementations from filter.module to plugin classes](/project/drupal/issues/3226806)

- language_get_default_langcode

- language_negotiation_url_prefixes_update

- language_get_browser_drupal_langcode_mappings

[#3574727: Deprecate remaining language.module code](/project/drupal/issues/3574727)

_media_get_add_url
[#3568124: Deprecate remaining functions in media.module](/project/drupal/issues/3568124)

- _media_library_views_form_media_library_after_build

- _media_library_media_type_form_submit

- _media_library_configure_form_display

- _media_library_configure_view_display

[#3570839: Deprecate remaining underscore functions in media_library.module](/project/drupal/issues/3570839)

_menu_link_content_update_path_alias
[#3570863: Deprecate _menu_link_content_update_path_alias](/project/drupal/issues/3570863)

- syslog_logging_settings_submit

- syslog_facility_list

[#3570235: Deprecate functions in syslog.module](/project/drupal/issues/3570235)

- system_sort_themes

- system_check_directory

[#3571172: Deprecate system_sort_themes() and system_check_directory() in system module](/project/drupal/issues/3571172)

- taxonomy_build_node_index

- taxonomy_delete_node_index

[#3570238: Deprecate remaining functions in taxonomy](/project/drupal/issues/3570238)

- views_ui_add_ajax_trigger

- views_ui_standard_display_dropdown

- views_ui_build_form_url

[#3035340: Deprecate core/modules/views_ui/admin.inc](/project/drupal/issues/3035340)

---

## 93. [block_theme_initialize() had been deprecated](https://www.drupal.org/node/3566783)

- **Node ID**: [3566783](https://www.drupal.org/node/3566783)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566782](https://www.drupal.org/node/3566782)

**Description:**

block_theme_initialize() has been deprecated. The logic has been moved to BlockHooks as a protected method and no public replacement has been provided.

---

## 94. [Entity type helper method to determine if the entity ID is integer](https://www.drupal.org/node/3566814)

- **Node ID**: [3566814](https://www.drupal.org/node/3566814)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566801](https://www.drupal.org/node/3566801)

**Description:**

A new EntityTypeInterface helper method has been introduced that allows to check if the entity ID is an integer. This was necessary because we are doing this check in a lot of point in the site...

```php
/**
- Checks if this entity type has an integer ID.
- * @return bool|null
- Returns:
- - TRUE if the entity type ID is defined as integer;
- - FALSE if the entity type ID is defined as string;
- - NULL if the entity type lacks an ID key.
   */
  public function hasIntegerId(): ?bool;
```

Other changes:

- DefaultHtmlRouteProvider::getEntityTypeIdKeyType() is deprecated. Even this is a protected method, there is a big chance that it's used by 3rd-parties extending DefaultHtmlRouteProvider, so we prefer to deprecate it instead of a simple remove.

- \Drupal\comment\CommentTypeForm::entityTypeSupportsComments() is deprecated.

- _comment_entity_uses_integer_id() is deprecated

---

## 95. [The comment_preview() function is deprecated and the logic has moved to CommentForm](https://www.drupal.org/node/3566882)

- **Node ID**: [3566882](https://www.drupal.org/node/3566882)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566879](https://www.drupal.org/node/3566879)

**Description:**

The comment_preview() was deprecated and the logic was moved inside CommentForm::preview()

---

## 96. [Functions providing a widget to enable content translation on bundle form are moved to a service](https://www.drupal.org/node/3566911)

- **Node ID**: [3566911](https://www.drupal.org/node/3566911)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566890](https://www.drupal.org/node/3566890)

**Description:**

The following functions are used to create a widget which could be placed on the bundle entity forms to enable content translation for those bundles, in the situation in which a language_configuration form element is not used:

- content_translation_enable_widget()

- content_translation_language_configuration_element_process()

- content_translation_language_configuration_element_validate()

- content_translation_language_configuration_element_submit()

The functions are deprecated and the functionality is converted to a new service: Drupal\content_translation\ContentTranslationEnableTranslationPerBundle

**Before:**

```php
class FooHooks {

  #[Hook('form_field_config_edit_form_alter')]
  public function formFieldConfigEditFormAlter(array &$form, FormStateInterface $form_state) : void {
    $form['lang'] = content_translation_enable_widget();
  }

}
```

**After:**

```php
use Drupal\content_translation\ContentTranslationEnableTranslationPerBundle;
use Drupal\Core\Hook\Attribute\Hook;

class FooHooks {

  public function __construct(
    protected ContentTranslationEnableTranslationPerBundle $contentTranslationWidget
  ) {}

  #[Hook('form_field_config_edit_form_alter')]
  public function formFieldConfigEditFormAlter(array &$form, FormStateInterface $form_state) : void {
    $form['lang'] = $this->contentTranslationWidget->getWidget('user', 'user', $form, $form_state);
  }

}
```

---

## 97. [Views::pluginManager() and Views::handlerManager() are deprecated](https://www.drupal.org/node/3566982)

- **Node ID**: [3566982](https://www.drupal.org/node/3566982)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566424](https://www.drupal.org/node/3566424)

**Description:**

The static methods Views::pluginManager() and Views::handlerManager() have been deprecated in favor of accessing Views plugin managers via dependency injection or using a new service locator.

Code that accessed Views plugin managers through static wrapper methods:

```php
use Drupal\views\Views;

// Get a plugin manager for a specific type.
$filter_manager = Views::pluginManager('filter');
$field_manager = Views::handlerManager('field');

// Use with dynamic types.
$type = 'display';
$manager = Views::pluginManager($type);
```

should now use the service directly, or the service locator for dynamic types:

```php
// For known/static types, use the specific service directly.
$filter_manager = \Drupal::service('plugin.manager.views.filter');
$field_manager = \Drupal::service('plugin.manager.views.field');

// For dynamic types, use the service locator.
$type = 'display';
$manager = \Drupal::service('views.plugin_managers')->get($type);
```

Similarly the service locator can be injected:

```php
use Psr\Container\ContainerInterface;

class MyService {
  public function __construct(
    #[Autowire(service: 'views.plugin_managers')]
    private readonly ContainerInterface $viewsPluginManagers,
  ) {}

  public function doSomething(string $type): void {
    $manager = $this->viewsPluginManagers->get($type);
    // ...
  }
}
```

---

## 98. [The Migrate Drupal module is deprecated](https://www.drupal.org/node/3566999)

- **Node ID**: [3566999](https://www.drupal.org/node/3566999)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3533584](https://www.drupal.org/node/3533584)

**Description:**

The Migrate Drupal, which provides source migrations from Drupal 6 and 7, is deprecated in Drupal 11 and will be removed in Drupal 12.

[Read](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-migrate-drupal) how this affects the Migrate API and the migration of legacy sites to Drupal 12.

---

## 99. [robots.txt blocks search pages with query parameters](https://www.drupal.org/node/3567198)

- **Node ID**: [3567198](https://www.drupal.org/node/3567198)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3550083](https://www.drupal.org/node/3550083)

**Description:**

The default robots.txt file now includes rules to prevent search engine crawlers from indexing search result pages that use query parameters. Both path-based and query parameter search URLs are blocked.

This prevents search engines from crawling URLs like /search?q=keyword, /search?f[0]=type:article, or any search result page that uses query strings.

Only sites with a customized robots.txt file should update the file to include these changes.

**Examples for sites with a custom robots.txt:**

Manually add these two lines to your robots.txt file:

**Monolingual sites:**

```php
Disallow: /search?
Disallow: /index.php/search?
```

**Multilingual sites:**

```php
Disallow: /*/search/
Disallow: /*/search?
```

---

## 100. [The content_translation_translate_access() and _content_translation_install_field_storage_definitions() functions are deprecated](https://www.drupal.org/node/3567484)

- **Node ID**: [3567484](https://www.drupal.org/node/3567484)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566890](https://www.drupal.org/node/3566890)

**Description:**

The content_translation_translate_access() function is deprecated and its logic was moved in the content_translation.manager service as a new ::access() method.

**Before:**

```php
$comment = \Drupal\comment\Entity\Comment::load(123);

if (content_translation_translate_access($comment)) {
  // Do something...
}

...

_content_translation_install_field_storage_definitions($settings->getTargetEntityTypeId());
```

**After:**

```php
$comment = \Drupal\comment\Entity\Comment::load(123);

// In a real case scenario properly inject the content_translation.manager service.
if (\Drupal::service('content_translation.manager')->access($comment)) {
  // Do something...
}

...

$this->installFieldStorageDefinitions($settings->getTargetEntityTypeId());
```

The protected method \Drupal\comment\CommentLazyBuilders::access(), which was only a wrapper around content_translation_translate_access() was also deprecated.

---

## 101. [FormattableMarkup: NULL @ and % placeholder values should log a notice and use empty string](https://www.drupal.org/node/3567614)

- **Node ID**: [3567614](https://www.drupal.org/node/3567614)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3554974](https://www.drupal.org/node/3554974)

**Description:**

As of Drupal 11.4.0, NULL values passed to @ and % placeholders in FormattableMarkup (and by extension TranslatableMarkup, logger calls, etc.) are now handled gracefully instead of throwing a fatal TypeError.

- @ placeholders (plain text): NULL triggers a PHP notice, outputs empty string

- % placeholders (emphasized): NULL triggers a PHP notice, outputs empty *tag*

- : placeholders (URLs): NULL is silently converted to empty string — no notice (intentional, as NULL is a common legitimate value for optional URLs)

The PHP notice includes the placeholder key and the source string to help identify where the bad value is coming from:

Passing NULL as a placeholder value is not supported. Use an empty string instead. Placeholder: %directory, String: "The upload directory %directory could not be created or is not accessible.".
**Action Required:**
No immediate action is required — your pages will no longer crash. However, the PHP notice will appear in your logs, and you should audit calling code to pass '' instead of NULL:

```php

---

## 102. [image_path_flush(), image_style_options() and IMAGE_DERIVATIVE_TOKEN have been deprecated](https://www.drupal.org/node/3567619)

- **Node ID**: [3567619](https://www.drupal.org/node/3567619)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3567618](https://www.drupal.org/node/3567618)

**Description:**

The constant IMAGE_DERIVATIVE_TOKEN has been deprecated.  Use ImageStyleInterface::TOKEN instead.

image_path_flush() and image_style_options() have both been deprecated.  See below for replacements.

**Before:**

```php
image_path_flush();
```

**After:**

```php
use Drupal\image\ImageDerivativeUtilities;

\Drupal::service(ImageDerivativeUtilities::class)->pathFlush();
```

**Before:**

```php
image_style_options();
```

**After:**

```php
use Drupal\image\ImageDerivativeUtilities;

\Drupal::service(ImageDerivativeUtilities::class)->styleOptions();
```

---

## 103. [The History module is deprecated](https://www.drupal.org/node/3567647)

- **Node ID**: [3567647](https://www.drupal.org/node/3567647)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3520472](https://www.drupal.org/node/3520472)

**Description:**

History is deprecated and will be removed from Drupal 12.0.0.

If you want to keep using the functionality provided by History, read the [recommendations for History](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-history).

---

## 104. [Deprecation of Drupal\Core\Theme\Registry::getBaseHook()](https://www.drupal.org/node/3567793)

- **Node ID**: [3567793](https://www.drupal.org/node/3567793)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2957444](https://www.drupal.org/node/2957444)

**Description:**

Drupal\Core\Theme\Registry::getBaseHook() has been deprecated. There is no replacement.

---

## 105. [Implementations of CategorizingPluginManagerInterface:: getSortedDefinitions() and :: getGroupedDefinitions() require a $labelKey argument](https://www.drupal.org/node/3567811)

- **Node ID**: [3567811](https://www.drupal.org/node/3567811)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3520730](https://www.drupal.org/node/3520730)

**Description:**

The signature of CategorizingPluginManagerInterface:: getSortedDefinitions() and :: getGroupedDefinitions() implementations will require a $labelKey argument in Drupal 12. This is to align the interface to the current real usage.

The $labelKey argument indicates the label to use for sorting/grouping plugins.

```php
public function getGroupedDefinitions(?array $definitions = NULL, string $label_key = 'label');
public function getSortedDefinitions(?array $definitions = NULL, string $label_key = 'label');
```

---

## 106. [Implementations of ExecutableInterface:: execute() require an $object argument](https://www.drupal.org/node/3567812)

- **Node ID**: [3567812](https://www.drupal.org/node/3567812)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3520730](https://www.drupal.org/node/3520730)

**Description:**

The signature of \Drupal\Core\Executable\ExecutableInterface:: execute() implementations will require a $object argument in Drupal 12. This is to align the interface to the current real usage.

The $object argument indicates the object to execute the plugin on/with.

```php
public function execute(?object $object = NULL);
```

---

## 107. [The long format 'filter tips' are deprecated](https://www.drupal.org/node/3567879)

- **Node ID**: [3567879](https://www.drupal.org/node/3567879)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3505370](https://www.drupal.org/node/3505370)

**Description:**

Before CKEditor was widely used, content authors were expected to enter HTML tags directly. To give hints to users about what formatting is allowed, text filters could provide tips in both short and long formats. The short format is displayed below the text area. An "About text formats" link takes the user to a page giving more information on how to write successful HTML with the long format tips.

Now most formatted text areas use CKEditor, the tips are no longer useful. The long format of the tips has been deprecated, along with the page that displays them.

Custom filters could previously implement both tip variants via the $long parameter:

```php
/**
- {@inheritdoc}
   */
  public function tips($long = FALSE) {
    if ($long) {
      return $this->t('Lines and paragraphs are automatically recognized. The <br /> line break, <p> paragraph and </p> close paragraph tags are inserted automatically. If paragraphs are not recognized simply add a couple of blank lines.');
    }
    else {
      return $this->t('Lines and paragraphs break automatically.');
    }
  }
```

The argument has been deprecated and only the short tip should be returned:

```php
/**
- {@inheritdoc}
   */
  public function tips() {
    return $this->t('Lines and paragraphs break automatically.');
  }
```

---

## 108. [Query parameters can be mapped directly to controller arguments](https://www.drupal.org/node/3567958)

- **Node ID**: [3567958](https://www.drupal.org/node/3567958)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3559628](https://www.drupal.org/node/3559628)

**Description:**

The #[MapQueryParameters] attribute from Symfony is now supported in Drupal. Controllers that access query string parameters can now declare them directly in the method signature instead of having to read them from the request.

Before:

```php
public function setDefaultTheme(Request $request) {
    $theme = $request->query->get('theme');
```

After:

```php
public function setDefaultTheme(#[MapQueryParameter] string $theme) {
```

Mapped arguments can be typed, validated, and nullable or provide default values where this is needed. See the [Symfony documentation](https://symfony.com/blog/new-in-symfony-6-3-query-parameters-mapper) for more details.

---

## 109. [The history module has been removed from the standard profile and recipe](https://www.drupal.org/node/3568078)

- **Node ID**: [3568078](https://www.drupal.org/node/3568078)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3567650](https://www.drupal.org/node/3567650)

**Description:**

The history module has been removed from the standard install profile and recipe, in preparation for being moved to a contributed module in Drupal 12.

Sites can continue to install history module when it's required.

---

## 110. [The _contextual_links_to_id() & _contextual_id_to_links() functions are deprecated](https://www.drupal.org/node/3568088)

- **Node ID**: [3568088](https://www.drupal.org/node/3568088)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3568087](https://www.drupal.org/node/3568087)

**Description:**

The _contextual_links_to_id() & _contextual_id_to_links() functions are deprecated and their functionality has been moved to a new Drupal\contextual\ContextualLinksSerializer service.

**Before:**

```php
$variables['title_suffix']['contextual_links'] = [
  '#type' => 'contextual_links_placeholder',
  '#id' => _contextual_links_to_id($element['#contextual_links']),
];
...
$element = [
  '#type' => 'contextual_links',
  '#contextual_links' => _contextual_id_to_links($id),
];
```

**After:**

```php
use use Drupal\contextual\ContextualLinksSerializer;
...
// Inject when possible.
$service = \Drupal::service(ContextualLinksSerializer::class);

$variables['title_suffix']['contextual_links'] = [
  '#type' => 'contextual_links_placeholder',
  '#id' => $service->linksToId($element['#contextual_links']),
];
...
$element = [
  '#type' => 'contextual_links',
  '#contextual_links' => $service->idToLinks($id),
];
```

---

## 111. [Underscore prefixed functions from editor.module are deprecated](https://www.drupal.org/node/3568136)

- **Node ID**: [3568136](https://www.drupal.org/node/3568136)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3568101](https://www.drupal.org/node/3568101)

**Description:**

The following functions from editor.module deprecated without a replacement and their functionality is moved, as protected methods, to \Drupal\editor\Hook\EditorHooks:

- _editor_record_file_usage() -&gt; EditorHooks::recordFileUsage()

- _editor_delete_file_usage() -&gt; EditorHooks::deleteFileUsage()

- _editor_get_file_uuids_by_field() -&gt; EditorHooks::getFileUuidsByField()

- _editor_get_formatted_text_fields() -&gt; EditorHooks::getFormattedTextFields()

- _editor_parse_file_uuids() -&gt; EditorHooks::parseFileUuids()

These functions will be removed from 12.0.0

---

## 112. [The editor_filter_xss() function is deprecated and functionality is moved to a service](https://www.drupal.org/node/3568146)

- **Node ID**: [3568146](https://www.drupal.org/node/3568146)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3568144](https://www.drupal.org/node/3568144)

**Description:**

The editor_filter_xss() function is deprecated and functionality is moved to a new method \Drupal\editor\Element::filters()

---

## 113. [Global constants in locale module are deprecated](https://www.drupal.org/node/3568156)

- **Node ID**: [3568156](https://www.drupal.org/node/3568156)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2831617](https://www.drupal.org/node/2831617)

**Description:**

Global constants from Local module are deprecated and replaced according to the following table:

Deprecated constant
Replaced by

**No public replacement:**

LOCALE_JS_STRING
moved as local variable to _locale_parse_js_file()

LOCALE_JS_OBJECT
moved as local variable to _locale_parse_js_file()

LOCALE_JS_OBJECT_CONTEXT
moved as local variable to_locale_parse_js_file()

LOCALE_NOT_CUSTOMIZED
FALSE

LOCALE_CUSTOMIZED
TRUE

LOCALE_TRANSLATION_OVERWRITE_ALL
moved as protected constant in LocaleSettingsForm

LOCALE_TRANSLATION_OVERWRITE_NON_CUSTOMIZED
moved as protected constant in LocaleSettingsForm

LOCALE_TRANSLATION_OVERWRITE_NONE
moved as protected constant in LocaleSettingsForm

LOCALE_TRANSLATION_STATUS_TTL
moved as protected constant in TranslationStatusForm

**Converted to enums:**

LOCALE_TRANSLATION_USE_SOURCE_LOCAL
\Drupal\locale\Model\UsedSourceType::Local

LOCALE_TRANSLATION_USE_SOURCE_REMOTE_AND_LOCAL
\Drupal\locale\Model\UsedSourceType::RemoteAndLocal

LOCALE_TRANSLATION_REMOTE
\Drupal\locale\Model\TranslationSource::Remote

LOCALE_TRANSLATION_LOCAL
\Drupal\locale\Model\TranslationSource::Local

LOCALE_TRANSLATION_CURRENT
\Drupal\locale\Model\TranslationSource::Current

---

## 114. [text_summary() is deprecated and moved to new TextSummary service](https://www.drupal.org/node/3568389)

- **Node ID**: [3568389](https://www.drupal.org/node/3568389)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3568387](https://www.drupal.org/node/3568387)

**Description:**

The text_summary function is deprecated and will be removed in Drupal 13. The functionality has moved to the TextSummary service.

**Before****text_summary($filtered_markup, 'neither_filter_enabled', 30);
After**

```php
use Drupal\text\TextSummary;
\Drupal::service(TextSummary::class)->generate($filtered_markup, 'neither_filter_enabled', 30);
```

---

## 115. [MigrationPluginManager no longer needs the language manager](https://www.drupal.org/node/3568857)

- **Node ID**: [3568857](https://www.drupal.org/node/3568857)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3568528](https://www.drupal.org/node/3568528)

**Description:**

The constructor for \Drupal\migrate\Plugin\MigrationPluginManager previously accepted a LanguageManagerInterface object as the last argument. The language manager is not used by the class, so the dependency and argument have been removed.

---

## 116. [Undocumented User::$password property is deprecated](https://www.drupal.org/node/3569185)

- **Node ID**: [3569185](https://www.drupal.org/node/3569185)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2648272](https://www.drupal.org/node/2648272)

**Description:**

RegisterForm is adding an undocumented password property on the $account object.
Accessing this property is deprecated and the property will be removed in Drupal 12.
Accessing this property will now trigger a deprecation:

```php
$password = $account->password;
```

---

## 117. [Usage of TRUNCATE ::execute() method is deprecated](https://www.drupal.org/node/3569194)

- **Node ID**: [3569194](https://www.drupal.org/node/3569194)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3569137](https://www.drupal.org/node/3569137)

**Description:**

Usage of TRUNCATE ::execute() method is deprecated: calling code should not rely on its return value. An exception will be thrown starting Drupal 13.

Use the ::executeSafe() method, that is not returning any value.

Before:

```php
public function deleteAll() {
    try {
      $return = $this->connection->truncate($this->bin)->execute();
    }
    catch ...
  }
```

After:

```php
public function deleteAll() {
    try {
      $this->connection->truncate($this->bin)->executeSafe();
    }
    catch ...
  }
```

---

## 118. [All code in locale.translations.inc has been deprecated](https://www.drupal.org/node/3569330)

- **Node ID**: [3569330](https://www.drupal.org/node/3569330)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3569328](https://www.drupal.org/node/3569328)

**Description:**

All of the content of locale.translations.inc have been deprecated.

**New service: LocaleSources:**

Before
After

locale_translation_load_sources
LocaleSource-&gt;loadSources

locale_translation_build_sources
LocaleSource-&gt;buildSources

locale_translation_source_check_file
LocaleSource-&gt;sourceCheckFile

locale_translation_source_build
LocaleSource-&gt;sourceBuild

locale_translation_build_server_pattern
LocaleSource-&gt;buildServerPattern

_locale_translation_file_is_remote
Protected

**New service LanguageDefaultOptions:**

This needs to be static where it is called.

Before
After

_locale_translation_default_update_options
LanguageDefaultOptions::updateOptions

**locale_translation_get_projects:**

**Important note:** in this CR locale_translation_get_projects was moved to LocaleProjectStorage
Before 11.4.0 LocaleProjectStorage was deprecated so the replacement was moved. See: [https://www.drupal.org/node/3037033](https://www.drupal.org/node/3037033)

Before
After

locale_translation_get_projects
\Drupal::service(LocaleProjectRepository::class)-&gt;getMultiple($project_names);

**Deprecate / remove without replacement:**

- locale_translation_clear_cache_projects -&gt; replace with local static cache property

- locale_cron_fill_queue -&gt; Logic inlined into LocaleHooks

- _locale_translation_source_compare -&gt;Logic simplified and inlined into TranslationStatusForm

- LOCALE_TRANSLATION_SOURCE_COMPARE_LT

- LOCALE_TRANSLATION_SOURCE_COMPARE_EQ

- LOCALE_TRANSLATION_SOURCE_COMPARE_GT

---

## 119. [Manual user creation now emails users by default](https://www.drupal.org/node/3569591)

- **Node ID**: [3569591](https://www.drupal.org/node/3569591)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3481627](https://www.drupal.org/node/3481627)

**Description:**

When manually creating a user account via the user creation form, email notification is now enabled by default using the option “Email user with password setup instructions”.

With this option selected, the password and status fields are hidden, reflecting that the user will complete account setup via the emailed instructions.

Administrators can disable the email option if they prefer to manually set the password and account status, in which case the form behaves as before.

---

## 120. [AJAX page state is now a request attribute](https://www.drupal.org/node/3569876)

- **Node ID**: [3569876](https://www.drupal.org/node/3569876)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3555532](https://www.drupal.org/node/3555532)

**Description:**

The ajax_page_state query parameter or request key holds information about previously loaded libraries, so that AJAX requests only add new libraries that are needed.

This is usually transparent and handled automatically, but following changes upstream in Symfony the ajax_page_state is added as a request attribute in early middleware. Any code that previously looked in the query string or request directly for the state should now look in the request attributes instead.

Previously:

```php
$ajax_page_state = $request->get('ajax_page_state');
// or
$ajax_page_state = $request->query->get('ajax_page_state');
```

Becomes:

```php
$ajax_page_state = $request->attributes->get('ajax_page_state');
```

---

## 121. [Standard profile and recipes no longer use text_with_summary](https://www.drupal.org/node/3569941)

- **Node ID**: [3569941](https://www.drupal.org/node/3569941)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3548756](https://www.drupal.org/node/3548756)

**Description:**

The Standard profile and recipes (those with content types) no longer use text_with_summary widget but instead use text_long.

Form displays use text_textarea instead of text_textarea_with_summary.

View displays use text_trimmed instead of text_summary_or_trimmed.

---

## 122. [Controller methods can now have autowired arguments](https://www.drupal.org/node/3570423)

- **Node ID**: [3570423](https://www.drupal.org/node/3570423)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3398528](https://www.drupal.org/node/3398528)

**Description:**

Controllers can already use services by wiring them into the controller, including via autowiring.

Controllers that only need a service in a single method can now have the service injected into the method arguments instead of the constructor.

Before:

```php
class MyController {
  public function __construct(protected ConfigFactoryInterface $configFactory) {}

  public function myPage() {
    $modules = $this->configFactory->get('core.extension');
    ...
  }
}
```

Now:

```php
class MyController {
  public function myPage(ConfigFactoryInterface $configFactory) {
    $modules = $configFactory->get('core.extension');
    ...
  }
}
```

In simple cases, the service can be inferred from the interface name alone. If a service cannot be determined from the interface directly, an exception will be raised, and you can add the #[Autowire] attribute to specify the service ID.

---

## 123. [The $configImporter property in tests is deprecated](https://www.drupal.org/node/3570442)

- **Node ID**: [3570442](https://www.drupal.org/node/3570442)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3560483](https://www.drupal.org/node/3560483)

**Description:**

KernelTestBase and BrowserTestBase provide a $this-&gt;configImporter property for testing config imports.

This property is now deprecated and will be removed in Drupal 13. Tests that want to test config imports should use the ConfigTestTrait and call $this-&gt;configImporter() when they need a config importer object.

---

## 124. [New config action to override static menu links](https://www.drupal.org/node/3570506)

- **Node ID**: [3570506](https://www.drupal.org/node/3570506)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3569949](https://www.drupal.org/node/3569949)

**Description:**

Some menu links are defined in code, by modules' MODULE.links.menu.yml files, or in hooks. Core has a service for overriding certain aspects of these links (such as their weight, and whether they are enabled or not). Recipes can now override such links with a config action:

```php
config:
    actions:
        core.menu.static_menu_link_overrides:
            overrideMenuLinks:
                some.link.id:
                    enabled: false
                    weight: 10
                some_other.link_id: null
```

Let's break this down:
The overrideMenuLinks config action can only be used on the special core.menu.static_menu_link_overrides config name. Trying to use the action on any other config name will throw an exception.

The action takes an array whose keys are menu link IDs, as they would be found in MODULE.menu.links.yml. The values are either an array of keys to override for that menu link (weight and enabled are probably the most useful, but see [https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Menu%21St...](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Menu%21StaticMenuLinkOverrides.php/function/StaticMenuLinkOverrides%3A%3AsaveOverride/11.x) for the full list), or NULL to undo any existing override for that link.

This action ONLY affects menu links defined in code. It will *not* work on menu links that are content entities, such as menu links for nodes or any custom menu link created via the UI.

---

## 125. [[Closed (outdated)]](https://www.drupal.org/node/3570512)

- **Node ID**: [3570512](https://www.drupal.org/node/3570512)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3570511](https://www.drupal.org/node/3570511)

**Description:**

REGIONS_VISIBLE and REGIONS_ALL global constants are deprecated and moved to an enum:

Deprecated constant
Replacement

REGIONS_VISIBLE
\Drupal\Core\Theme\ShowRegions::Visible

REGIONS_ALL
\Drupal\Core\Theme\ShowRegions::All

---

## 126. [SessionManager::delete() is deprecated](https://www.drupal.org/node/3570851)

- **Node ID**: [3570851](https://www.drupal.org/node/3570851)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3570849](https://www.drupal.org/node/3570849)

**Description:**

SessionManager::delete() is deprecated in Drupal 11.4.0 and is removed from Drupal 13.0.0. Use Drupal\Core\Session\UserSessionRepository::deleteAll() instead.

**Before:**

```php
public function postSave(EntityStorageInterface $storage, $update = TRUE) {
    parent::postSave($storage, $update);

    if ($update) {
      $session_manager = \Drupal::service('session_manager');
      // If the password has been changed, delete all open sessions for the
      // user and recreate the current one.
      if ($this->pass->value != $this->getOriginal()->pass->value) {
        $session_manager->delete($this->id());
        // [...]
}
```

**After:**

```php
public function postSave(EntityStorageInterface $storage, $update = TRUE) {
    parent::postSave($storage, $update);

    if ($update) {
      $session_repository = \Drupal::service(UserSessionRepositoryInterface::class);
      // If the password has been changed, delete all open sessions for the
      // user and recreate the current one.
      if ($this->pass->value != $this->getOriginal()->pass->value) {
        $session_repository->deleteAll($this->id());
        // [...]
}
```

---

## 127. [Cache metadata for computed fields is now bubbled for JSON:API responses](https://www.drupal.org/node/3570884)

- **Node ID**: [3570884](https://www.drupal.org/node/3570884)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3252278](https://www.drupal.org/node/3252278)

**Description:**

If a computed field value is returned via JSON:API, the computed field item list class can now implement Drupal\Core\Cache\CacheableDependencyInterface in order to have its cacheability metadata bubble to the response.

This was already fixed for rendered output in 10.2.4 (see [https://www.drupal.org/node/3423720](https://www.drupal.org/node/3423720)) and is now fixed for JSON:API.

As is noted in the other change notice, note that theoretically the added logic applies to the field item list classes of non-computed fields, as well, but it generally does not make sense to apply cacheable metadata to non-computed fields.

---

## 128. [The editor_image_upload_settings_form() is deprecated](https://www.drupal.org/node/3570919)

- **Node ID**: [3570919](https://www.drupal.org/node/3570919)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3570917](https://www.drupal.org/node/3570917)

**Description:**

- The editor_image_upload_settings_form() is deprecated. Its logic is moved to a service: \Drupal\editor\EditorImageUploadSettingsTrait::getForm()

- The whole core/modules/editor/editor.admin.inc file is deprecated.

**Before:**

```php
class Image extends CKEditor5PluginDefault implements CKEditor5PluginConfigurableInterface {
  ...
  public function buildConfigurationForm(array $form, FormStateInterface $form_state) {
    $form_state->loadInclude('editor', 'admin.inc');
    return editor_image_upload_settings_form($form_state->get('editor'));
  }
  ...
}
```

**After:**

```php
class Image extends CKEditor5PluginDefault implements CKEditor5PluginConfigurableInterface {
  public function __constructor() {
    ...
    protected EditorImageUploadSettings $editorImageUploadSettings,
  }
  ...
  public function buildConfigurationForm(array $form, FormStateInterface $form_state) {
    return $this->editorImageUploadSettings->getForm($form_state->get('editor'));
  }
  ...
}
```

---

## 129. [Functions in menu_ui.module are deprecated and move to hooks](https://www.drupal.org/node/3571402)

- **Node ID**: [3571402](https://www.drupal.org/node/3571402)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3571400](https://www.drupal.org/node/3571400)

**Description:**

All the functions in menu_ui.module have been deprecated

Before
After

_menu_ui_node_save
MenuUiHelper::menuUiNodeSave()

menu_ui_get_menu_link_defaults()
MenuUiHelper::getMenuLinkDefaults()

menu_ui_node_builder
\Drupal::service(MenuUiHooks::class)-&gt;nodeBuilder()

menu_ui_form_node_form_submit
\Drupal::service(MenuUiHooks::class)-&gt;formNodeFormSubmit()

menu_ui_form_node_type_form_validate
\Drupal::service(MenuUiHooks::class)-&gt;formNodeTypeFormValidate()

menu_ui_form_node_type_form_builder
\Drupal::service(MenuUiHooks::class)-&gt;formNodeTypeFormBuilder()

---

## 130. [locale.settings:translation.path config is deprecated in favor of locale_translation_path setting](https://www.drupal.org/node/3571594)

- **Node ID**: [3571594](https://www.drupal.org/node/3571594)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3571593](https://www.drupal.org/node/3571593)

**Description:**

The configuration setting is deprecated and can no longer be changed in the UI.

Set $settings['locale_translation_path'] = '/the/path' instead.

The update will remove and notify about a customized value.

---

## 131. [The Contact module is deprecated](https://www.drupal.org/node/3571602)

- **Node ID**: [3571602](https://www.drupal.org/node/3571602)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3520470](https://www.drupal.org/node/3520470)

**Description:**

The Contact module is deprecated and will be removed from Drupal 12.0.0.

If you want to keep using the functionality provided by Contact, read the [recommendations for Contact](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-contact).

---

## 132. [Translation interface adds context/location filters and new StringContextInterface](https://www.drupal.org/node/3572228)

- **Node ID**: [3572228](https://www.drupal.org/node/3572228)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2123543](https://www.drupal.org/node/2123543)

**Description:**

A new StringContextInterface has been added to support filtering translatable strings by context and location in the translation interface (/admin/config/regional/translate).
**User interface changes:**
The translate interface UI now includes "Context" and "Location" filter dropdowns.
**API changes:**
A new optional interface:

```php
namespace Drupal\locale;

interface StringContextInterface {
  public function getContexts(): array;
  public function getDistinctLocations(): array;
}
```

StringDatabaseStorage now implements this interface.
**Example:**

```php
$locale_storage = \Drupal::service('locale.storage');

// Get available contexts and locations.
if ($locale_storage instanceof \Drupal\locale\StringContextInterface) {
  $contexts = $locale_storage->getContexts();
  $locations = $locale_storage->getDistinctLocations();
}
```

**Backward compatibility:**
No BC break. This is a new optional interface.

---

## 133. [Migration plugins link_options, link_uri, timezone, and user_langcode are moved to the Migrate module](https://www.drupal.org/node/3572239)

- **Node ID**: [3572239](https://www.drupal.org/node/3572239)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3560075](https://www.drupal.org/node/3560075)

**Description:**

Four classes are deprecated in Drupal 11.4.0, to be removed in Drupal 13.0.0:

- Drupal\menu_link_content\Plugin\migrate\process\LinkOptions

- Drupal\menu_link_content\Plugin\migrate\process\LinkUri

- Drupal\system\Plugin\migrate\process\d6\Timezone

- Drupal\user\Plugin\migrate\process\UserLangcode

They are replaced by equivalent classes in the Migrate module:

- Drupal\migrate\Plugin\migrate\process\LinkOptions

- Drupal\migrate\Plugin\migrate\process\LinkUri

- Drupal\migrate\Plugin\migrate\process\Timezone

- Drupal\migrate\Plugin\migrate\process\UserLangcode

(The old Timezone class is in the d6 namespace, but the new one is not.)

**Migrations:**

There is no change for migrations, since these process plugins are referenced by their plugin IDs, which have not changed:

- link_uri

- link_options

- timezone

- user_langcode

The plugin IDs now reference the new classes in the Migrate module.

**PHP Code:**

If you refer to these classes by their fully qualified class names, then you will have to update the namespace. This is most common in classes that extend the core plugin classes. For example, suppose you have a custom version of the timezone plugin. Then you will have to update the use statement in your module:

**Before Drupal 11.4.0:**

```php
namespace Drupal\mymodule\Plugin\migrate\process;

use Drupal\system\Plugin\migrate\process\d6\TimeZone;

/**
- Custom variant of the timezone process plugin.
 */
#[MigrateProcess('my_timezone')]
class MyTimeZone extends TimeZone {
  // Your code goes here.
}
```

**In Drupal 11.4.0 or later:**

```php
namespace Drupal\mymodule\Plugin\migrate\process;

use Drupal\migrate\Plugin\migrate\process\TimeZone;

/**
- Custom variant of the timezone process plugin.
 */
#[MigrateProcess('my_timezone')]
class MyTimeZone extends TimeZone {
  // Your code goes here.
}
```

---

## 134. [All functions in locale.fetch.inc are deprecated](https://www.drupal.org/node/3572345)

- **Node ID**: [3572345](https://www.drupal.org/node/3572345)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3572339](https://www.drupal.org/node/3572339)

**Description:**

All functions in locale.fetch.inc are deprecated and moved to the LocaleFetch service class.

locale_translation_batch_update_build is replaced by \Drupal::service(LocaleFetch::class)-&gt;buildUpdateBatch($projects, $langcodes, $options)

locale_translation_batch_fetch_build is replaced by \Drupal::service(LocaleFetch::class)-&gt;buildFetchBatch($projects, $langcodes, $options)

_locale_translation_fetch_operations -&gt; There is no replacement.

**Important note:** Before 11.4 was released the method names were changed: [https://www.drupal.org/node/3589759](https://www.drupal.org/node/3589759)

---

## 135. [Several Views functions have been deprecated](https://www.drupal.org/node/3572594)

- **Node ID**: [3572594](https://www.drupal.org/node/3572594)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3572243](https://www.drupal.org/node/3572243)

**Description:**

The following views functions have been deprecated.

Many functions have a trivial OOP replacement. Function to get the current view have been deprecated without replacement. Relying on a global current view is unreliable when fibers and caching is involved. As a possible alternative, when rendering entities within a view, it may be possible to use $entity-&gt;view. However, this will also not automatically provide the necessary cache contexts and is discouraged and may be removed in the future as well. Whenever possibly, rendered entities should only vary based on their view mode.

- views_view_is_enabled

- views_view_is_disabled

- views_enable_view

- views_disable_view

- views_embed_view

- views_set_current_view

- views_get_current_view

- views_get_view_result

- _views_query_tag_alter_condition

- views_element_validate_tags

Before
After

views_view_is_enabled
$view-&gt;status()

views_view_is_disabled
!$view-&gt;status()

views_enable_view
$view-&gt;enable()-&gt;save()

views_disable_view
$view-&gt;disable()-&gt;save()

views_embed_view

```php
[
    '#type' => 'view',
    '#name' => $name,
    '#display_id' => $display_id,
    '#arguments' => $args,
  ]
```

views_get_view_result
Views::getViewResult($name, $display_id, ...$args)

views_set_current_view
No replacement

views_get_current_view
No replacement

_views_query_tag_alter_condition
No replacement

views_element_validate_tags
No replacement

---

## 136. [Site information form now stores unresolved path aliases for front, 403, and 404 pages](https://www.drupal.org/node/3572707)

- **Node ID**: [3572707](https://www.drupal.org/node/3572707)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#1503146](https://www.drupal.org/node/1503146)

**Description:**

Previously, when you set the front page, 403 page, or 404 page on /admin/config/system/site-information, Drupal would convert path aliases to internal paths before saving. For example, if you entered /home (an alias for /node/1), the config stores /node/1.

This could cause redirects to /home instead of / when the front page is an aliased path, and it tied configuration to internal paths and thus to specific content (e.g. node IDs).

As of [#1503146: Aliased paths cannot be set as front page](/project/drupal/issues/1503146), the front page and error pages now save exactly what you type, regardless of whether it is an alias or a system path. If you enter /home, that's what will be stored in config. The issue introduces a new path matcher that fixes the redirection problem -- in our example, visit / will no longer redirect to /home.

The main "gotcha" here is that, if you set the front page to /home (a.k.a. node/1) and then you change node 1's path alias to something else, your home page will suddenly be a 404, and you will need to remember to update it in site settings.

The same idea applies to custom 403 and 404 paths.

---

## 137. [JSON:API normalisation now skips cacheing if a ResourceObject has max-age 0](https://www.drupal.org/node/3573370)

- **Node ID**: [3573370](https://www.drupal.org/node/3573370)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3572098](https://www.drupal.org/node/3572098)

**Description:**

It is now possible to skip Normalization cache for ResourceObject by setting the max-age to 0.

---

## 138. [file_get_file_references() is deprecated in favor of the FileReferenceResolver](https://www.drupal.org/node/3573884)

- **Node ID**: [3573884](https://www.drupal.org/node/3573884)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#1452100](https://www.drupal.org/node/1452100)

**Description:**

file_get_file_references() and file_field_find_file_reference_column() have been deprecated.

Replacing file_get_file_references() requires a review of the information below to understand the historical bugs and features fixed with this change. file_field_find_file_reference_column() has no replacement.

The file_get_file_references() function had bugs around usages in non-default revisions and very confusing DX with multiple arguments that needed to be set to a non-default value to get at least partially working results.

The function by default only returned usages of file fields (and not image and possibly others) and it didn't return anything if the use only existed on a non-default revision of an entity (either old or pending revisions), which meant that private files on those revisions were not accessible to users that should have access to them. Even if explicitly asked to return revisions with the use of the FIELD_LOAD_REVISION $age parameter.

The new API will yield FileReferenceUsage value objects instead of returning nested arrays. It will always and automatically return non-default revisions if the usage is not found on the default revision and exists on another. Only a single revision is returned for each entity, with the assumption that access does not vary between revisions, the user either has access to all or none.

There are some safeguards in place to limit the performance cost in edge cases, specifically the API will not return more than 50 non-default revisions per entity type. This is extremely unlikely to reach, as private files are rarely reused directly on many entities. It would also only result in users not having access if they do not have access to any of those first 50 entity revisions. Like before, no limit is put on the number of default revisions that are returned.

The existing API function does not call the new API due to fundamental API changes and the bug is not fixed, the new API must be used to support non-default revisions correctly.

**Before:**

```php
if ($references = file_get_file_references($file, NULL, EntityStorageInterface::FIELD_LOAD_REVISION, NULL)) {
        foreach ($references as $field_name => $entity_map) {
          foreach ($entity_map as $referencing_entities) {
            /** @var \Drupal\Core\Entity\EntityInterface $referencing_entity */
            foreach ($referencing_entities as $referencing_entity) {
              $entity_and_field_access = $referencing_entity->access('view', $account, TRUE)->andIf($referencing_entity->$field_name->access('view', $account, TRUE));
              if ($entity_and_field_access->isAllowed()) {
                return $entity_and_field_access;
              }
            }
          }
       }
    }
```

**After:**

```php
$resolver = \Drupal::service(FileReferenceResolver::class);
      foreach ($resolver->getReferences($entity) as $usage) {
        $has_references = TRUE;
        $referencing_entity = $resolver->loadEntityFromUsage($usage);

        // Either check view or revision access depending on this being a
        // default revision or not.
        if ($referencing_entity instanceof RevisionableInterface && !$referencing_entity->isDefaultRevision()) {
          $entity_and_field_access = $referencing_entity->access('view revision', $account, TRUE);
        }
        else {
          $entity_and_field_access = $referencing_entity->access('view', $account, TRUE);
        }

        // If access to that entity is allowed, check field access as well,
        // if access is still allowed, return this result.
        if ($entity_and_field_access->isAllowed()) {
          $entity_and_field_access = $entity_and_field_access->andIf($referencing_entity->get($usage->fieldName)->access('view', $account, TRUE));
          if ($entity_and_field_access->isAllowed()) {
            return $entity_and_field_access;
          }
        }
      }
```

Notes:

- The API changes only apply to custom usages of directly verifying file references, for the purpose of access checks or something else. Default private file routes now work out of the box with non-default revisions as well as when using $file-&gt;access('download'), no changes are necessary to benefit from the bugfix

- When using references for the purpose of access checks, the caller is responsible to check whether the entity is a default revision or not and check the appropriate access operation. The new implementation additionally optimizes the access check to only check field access if entity access is allowed, which results in some extra complexity

- Note: This does not address more complex use cases where private files are used by intermediate entity types such as medias, which in turn may be used by many other entities. Drupal core will continue to evaluate access only to the entity directly referencing the file. See [#2984093: Inform users that media items don't inherit access control from parents](/project/drupal/issues/2984093)

- If results really should be limited to a specific file type, then the caller is responsible to check that in the loop with $referencing_entity-&gt;getFieldDefinition($usage-&gt;fieldName)-&gt;getType()

---

## 139. [Markup for Toolbar buttons is changed to improve accessibility](https://www.drupal.org/node/3573894)

- **Node ID**: [3573894](https://www.drupal.org/node/3573894)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3093378](https://www.drupal.org/node/3093378)

**Description:**

**Summary of changes:**

This change affects the markup for the &lt;button&gt; elements in the admin toolbar when it is in vertical mode, such as the buttons next to "Structure" and "Display modes" in this screenshot:

There should not be any visible difference before or after this change, so the screenshot works for both.

**Before:**

Markup for the collapsed Structure item:

```php
**Extend
  Structure

```

Markup for the expanded Structure item:

```php

  Collapse
  Structure

```

**After:**

Markup for the collapsed Structure item:

```php

  Structure menu

```

Markup for the expanded Structure item:

```php

  Structure menu

```

**What themes are affected?:**

Any theme that uses the Twig templates, CSS, and JavaScript from the core Toolbar module will get the updated markup. That includes the core Claro, Olivero, Stark, and Starterkit theme themes.

Any theme that overrides the core Toolbar module will not** get the updated markup. That includes the core Stable 9 theme.

**Custom themes:**

**How to keep the old markup:**

If the new markup causes problems with your custom theme, then you can override the Toolbar module. The Stable 9 theme does this:

From stable9.info.yml:

```php
libraries-override:
  toolbar/toolbar:
    css:
      theme:
        css/toolbar.theme.css: css/toolbar/toolbar.theme.css
        css/toolbar.icons.theme.css: css/toolbar/toolbar.icons.theme.css
  toolbar/toolbar.menu:
    js:
      js/toolbar.menu.js: js/toolbar/toolbar.menu.js
```

**How to get the new markup:**

If your theme extends the Stable 9 theme, but you want to get the accessibility improvements from the new markup, then you will have to override the overrides. Add something like this to your theme's .info.yml file:

```php
libraries-override:
  # Use the CSS and JS files from the toolbar module, not the stable9 theme.
  toolbar/toolbar:
    css:
      theme:
        /core/themes/stable9/css/toolbar/toolbar.icons.theme.css: /core/modules/toolbar/css/toolbar.icons.theme.css
        /core/themes/stable9/css/toolbar/toolbar.theme.css: /core/modules/toolbar/css/toolbar.theme.css
  toolbar/toolbar.menu:
    js:
      /core/themes/stable9/js/toolbar/toolbar.menu.js: /core/modules/toolbar/js/toolbar.menu.js
```

---

## 140. [TestRequirementsTrait is deprecated](https://www.drupal.org/node/3574112)

- **Node ID**: [3574112](https://www.drupal.org/node/3574112)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3573954](https://www.drupal.org/node/3573954), [#3589047](https://www.drupal.org/node/3589047), [#3590435](https://www.drupal.org/node/3590435)

**Description:**

Drupal\Tests\TestRequirementsTrait has been subject to deprecations and removals in last majors, and is now deprecated itself. The ::getDrupalRoot() method is also deprecated.

Calling the ::getDrupalRoot() method is no longer necessary, tests extending Drupal base classes can access directly $this-&gt;root that is automatically initialized during the test set-up.

Example

Before:

```php
$this->root = static::getDrupalRoot();
    chdir($this->root);
```

After:

```php
chdir($this->root);
```

---

## 141. ["Manage display" now defaults to a display-builder agnostic overview page](https://www.drupal.org/node/3574383)

- **Node ID**: [3574383](https://www.drupal.org/node/3574383)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3564234](https://www.drupal.org/node/3564234)

**Description:**

A new overview page has been added at /admin/structure/types/manage/{bundle}/display. The "Manage display" tab now defaults to this overview instead of the default view mode edit form. The page lists all display modes for the bundle with their label, description, and enabled/disabled status, which can be toggled directly from the overview.

The previous route pointed directly to the default view mode edit form, which prevents deprecation of the default view mode ([#2844203: Improve/Simplify situation around Default/Full view modes/view displays](/project/drupal/issues/2844203)). The old interface also implicitly assumed Field UI is the only display builder, making it difficult for Layout Builder, Drupal Canvas, and other builders to integrate cleanly. The new overview page is display-builder agnostic and serves as a neutral navigation hub.

If your module has functional tests or code that relies on the "Manage display" tab routing directly to the default view mode, update those references to account for the new overview page.

---

## 142. [Select query objects now provide getRange() method](https://www.drupal.org/node/3574425)

- **Node ID**: [3574425](https://www.drupal.org/node/3574425)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2986699](https://www.drupal.org/node/2986699)

**Description:**

A new Select::getRange() method has been added to the database query API. This allows retrieving the currently configured range (start and length) from a Select query object.

Previously, there was no supported way to inspect the range applied to a query after calling range(). This made it difficult for query extenders and integrations to adjust behavior based on the existing range configuration.

The following methods were added:

```php
\Drupal\Core\Database\Query\SelectInterface::getRange()

\Drupal\Core\Database\Query\Select::getRange()

\Drupal\Core\Database\Query\SelectExtender::getRange()
```

The method returns an array with start and length keys, or NULL if no range is set.

**Before/After code examples:**

```php
$query = $connection->select('node', 'n');
$query->range(0, 10);

// No supported way to retrieve the range.
```

```php
$query = $connection->select('node', 'n');
$query->range(0, 10);

$range = $query->getRange();
// ['start' => 0, 'length' => 10]
```

---

## 143. [Layout Builder storage plugins must implement SupportAwareSectionStorageInterface](https://www.drupal.org/node/3574738)

- **Node ID**: [3574738](https://www.drupal.org/node/3574738)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3574461](https://www.drupal.org/node/3574461), [#3060985](https://www.drupal.org/node/3060985)

**Description:**

To optimize performance, section storage plugins must support \Drupal\layout_builder\SupportAwareSectionStorageInterface

Be aware that section storage plugins not working with entities, such as navigation.module,  can unconditionally return FALSE.

```php
/**
- {@inheritdoc}
   */
  public function isSupported(string $entity_type_id, string $bundle, string $view_mode): bool {
    // Navigation section storage does not support any entity type.
    return FALSE;
  }
```

---

## 144. [Entity bundle classes can be defined and discovered using the Drupal\Core\Entity\Attribute\Bundle attribute](https://www.drupal.org/node/3574891)

- **Node ID**: [3574891](https://www.drupal.org/node/3574891)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3301682](https://www.drupal.org/node/3301682)

**Description:**

The Drupal\Core\Entity\Attribute\Bundle attribute can be added to EntityInterface classes to designate the class as a [bundle class](https://www.drupal.org/node/3191609). In order for the bundle class to be automatically discovered, it must be in the \Entity namespace of a module and have the Bundle attribute.

The use of the attribute can be supplemental to, or in lieu of, defining bundle classes in entity_type_info or entity_type_info_alter hook implementations. The bundle class definitions discovered through attributes are added after entity_type_info implementations run and before entity_type_info_alter implementations. In the case attribute-defined bundle classes defined in multiple modules all target the same entity bundle, entity_type_info_alter implementations can be used to specify a different bundle class other than the one attribute discovery provides.

**Usage****hook_entity_type_info()

```php
#[Hook('entity_bundle_info')]
  public function entityBundleInfo(): array {
    $bundles['some_entity']['bundle_class_a']['class'] = EntityTestBundleClassA::class;
    $bundles['some_entity']['bundle_class_a']['label'] = 'Bundle class A';
    $bundles['some_entity']['bundle_class_a']['translatable'] = TRUE;

    return $bundles;
  }
```

Attribute****Example of an using attribute override existing bundle class from hook_entity_bundle_info above:

```php
#[Bundle(
  entityType: 'some_entity',
  bundle: 'bundle_class_a',
  label: new TranslatableMarkup('Bundle class A label set by attribute'),
  translatable: FALSE,
)]
class BundleClassOverrideA extends SomeEntity {}
```

The label and translatable properties are optional and will override the existing values only if the bundle properties have non-NULL values.

Example of creating a bundle class for entity types without a bundle entity type:**

```php
#[Bundle(
  entityType: 'some_entity',
  bundle: 'new_bundle',
  label: new TranslatableMarkup('New bundle'),
)]
class NewBundle extends SomeEntity {}
```

If creating a new bundle, setting the label explicitly is recommended, otherwise the bundle name will be used as the label.

Note that it is not possible to create a bundle class for a bundle (new_bundle in this example) for any entity type with a bundle entity type (such as node with node_type config entities). Bundle class definition for non-existing bundles will not be exposed as a new bundle. This is done to ensure that code that expects to be able to load the corresponding config entity for such a bundle to work consistently.

Such definitions are ignored instead of triggering an error to handle edge cases when adding or deleting those config entities.

---

## 145. ['uri_callback' entity key is deprecated](https://www.drupal.org/node/3575062)

- **Node ID**: [3575062](https://www.drupal.org/node/3575062)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2667040](https://www.drupal.org/node/2667040)

**Description:**

URI callbacks allowed entities to specify custom paths for canonical entity URLs. This feature has since been superseded by [link templates](https://www.drupal.org/docs/drupal-apis/entity-api/link-templates) which allows entities to have different paths for more than just the canonical route; they also allow paths to both be generated and looked up in a more consistent way.

URI callbacks are now deprecated in Drupal 11.4 and will be removed in Drupal 13. Use link templates where possible, or an outbound path processor if you need fine grained control over individual URLs.

---

## 146. [The shortcut module has been removed from the standard profile and recipe](https://www.drupal.org/node/3575093)

- **Node ID**: [3575093](https://www.drupal.org/node/3575093)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3569133](https://www.drupal.org/node/3569133)

**Description:**

The shortcut module has been removed from the standard install profile and recipe, in preparation for being moved to a contributed module in Drupal 12.

Sites can continue to install shortcut module when it's required.

---

## 147. [FormBase provides create() factory method with autowired parameters](https://www.drupal.org/node/3575184)

- **Node ID**: [3575184](https://www.drupal.org/node/3575184)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3398534](https://www.drupal.org/node/3398534)

**Description:**

Classes that implement \Drupal\Core\DependencyInjection\ContainerInjectionInterface must provide a create() factory method to instantiate the object.

Now that [services can be autowired](https://www.drupal.org/node/3218156) and [plugins can be autowired](https://www.drupal.org/node/3542837), the FormBase class has also been extended with AutowireTrait, providing a generic version of the create() method suitable for most forms that need dependency injection.

Previously, a constructor and create() method would be something like this:

```php
public function __construct(EntityDisplayRepositoryInterface $entity_display_repository, ConfigFactoryInterface $config_factory) {
    $this->entityDisplayRepository = $entity_display_repository;
    $this->configFactory = $config_factory;
  }

  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('entity_display.repository'),
      $container->get('config.factory')
    );
  }
```

Now it is possible to drop the create() method:

```php
public function __construct(EntityDisplayRepositoryInterface $entity_display_repository, ConfigFactoryInterface $config_factory) {
    $this->entityDisplayRepository = $entity_display_repository;
    $this->configFactory = $config_factory;
  }
```

In simple cases, the service can be inferred from the interface name alone. If a service cannot be determined from the interface directly, an exception will be raised, and you can add the #[Autowire] attribute to specify the service ID.

---

## 148. [AutowireTrait and AutowiredInstanceTrait support container parameters](https://www.drupal.org/node/3575335)

- **Node ID**: [3575335](https://www.drupal.org/node/3575335)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3558292](https://www.drupal.org/node/3558292)

**Description:**

Container parameters can be passed as autowired parameters to the constructors of classes using AutowireTrait and AutowiredInstanceTrait. These include classes implementing ContainerInjectionInterface, such as controllers and forms, and classes implementing ContainerFactoryPluginInterface, such as plugins. Previously, AutowireTrait and AutowiredInstanceTrait only supported autowiring services.

Controller example:

```php
use Drupal\Core\Controller\ControllerBase;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

class SampleController extends ControllerBase {

  public function __construct(
    #[Autowire(param: 'container.namespaces')]
    protected readonly array $namespaces,
  ) {}

}
```

Alternative:

```php
use Drupal\Core\Controller\ControllerBase;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

class SampleController extends ControllerBase {

  public function __construct(
    #[Autowire('%container.namespaces%')]
    protected readonly array $namespaces,
  ) {}

}
```

---

## 149. [The LinkFormatter constructor now requires an AccessManagerInterface argument](https://www.drupal.org/node/3575906)

- **Node ID**: [3575906](https://www.drupal.org/node/3575906)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3231247](https://www.drupal.org/node/3231247)

**Description:**

Drupal\link\Plugin\Field\FieldFormatter\LinkFormatter::__construct() has a new required argument: $accessManager of type Drupal\Core\Access\AccessManagerInterface.

If you instantiate this class directly or extend it, update your constructor call to inject the access_manager service.

---

## 150. [The 'version' value in .info.yml files must be a string](https://www.drupal.org/node/3576311)

- **Node ID**: [3576311](https://www.drupal.org/node/3576311)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3565033](https://www.drupal.org/node/3565033)

**Description:**

The version value in .info.yml files must be a string. Previously, using numeric literals was possible, but this can lead to all sorts of problems. For example, the version 1.0 would be parsed as a number, and simplified to just 1. To ensure consistency, the version value must now explicitly be a string wrapped in single quotes.

Note that extension developers distributing code on Drupal.org do not need to worry about this. The Drupal.org packaging system automatically adds the version key  to .info.yml files, and has always done so as a string literal. This change primarily impacts the .info.yml files for custom extensions.

**Before:**

An custom_example.info.yml file might look like this:

```php
core_version_requirement: '^10 || ^11'
name: 'Custom example'
type: module
version: 1.1
```

**After:**

Now, you must wrap the version value in quotes:

```php
core_version_requirement: '^10 || ^11'
name: 'Custom example'
type: module
version: '1.1'
```

---

## 151. [Accessing the autoload global is deprecated](https://www.drupal.org/node/3576336)

- **Node ID**: [3576336](https://www.drupal.org/node/3576336)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3313404](https://www.drupal.org/node/3313404)

**Description:**

To access Composer's autoloader (class loader), find and require /vendor/autoload.php.

Before, the class loader was available from the $autoload variable. This made it available as global variable, $GLOBALS['autoload'].

Accessing any methods on the autoload global will emit a deprecation error in Drupal 11.4.0 and be unsupported in Drupal 12.0.0.

---

## 152. [Locale now uses file hash instead of mtime to detect translation file changes](https://www.drupal.org/node/3576427)

- **Node ID**: [3576427](https://www.drupal.org/node/3576427)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3575678](https://www.drupal.org/node/3575678)

**Description:**

The Locale module previously used file modification timestamps (filemtime()) to determine whether a local .po translation file had changed and needed to be re-imported. This approach is unreliable in modern deployment workflows — most notably when Docker COPY commands are used to add translation files to an image, because Docker does not preserve file mtimes.

The module now computes a **xxHash (xxh128)** of each local translation file and stores it in a new hash column on the {locale_file} database table. Change detection for local files is based on hash comparison rather than timestamp comparison.

For **remote translation files**, the behaviour is unchanged — timestamp comparison using the HTTP Last-Modified header is still used, as no local file content is available before download.

---

## 153. [CachePluginBase::cacheExpire in views module is deprecated](https://www.drupal.org/node/3576855)

- **Node ID**: [3576855](https://www.drupal.org/node/3576855)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3576556](https://www.drupal.org/node/3576556)

**Description:**

This method was used in \Drupal\views\Plugin\views\cache\CachePluginBase::cacheGet() to ensure that existing caches were not expired.

The cache expiration is also set in \Drupal\views\Plugin\views\cache\CachePluginBase::cacheSet() based on \Drupal\views\Plugin\views\cache\CachePluginBase::cacheSetMaxAge(), the cache system will not return expired results.

It is safe to remove calls to cacheExpire() and remove checking this in any core version, the logic in cacheGet() was unnecessary at least since Drupal 8.0.

---

## 154. [New LocaleFile and LocaleFileManager](https://www.drupal.org/node/3577675)

- **Node ID**: [3577675](https://www.drupal.org/node/3577675)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3577671](https://www.drupal.org/node/3577671)

**Description:**

Several functions related to locale file management have been deprecated and replaced with the LocaleFile value object and LocaleFileManager service.

**Before****locale_translation_download_source($source_file, $directory)
After****\Drupal::service(LocaleFileManager::class)-&gt;downloadTranslationSource($source_file, $directory)'temporary://')
Before****locale_translate_get_interface_translation_files($projects, $langcodes)
After**

```php
\Drupal::service(LocaleFileManager::class)
->getInterfaceTranslationFiles($projects, $langcodes)
```

**Before****locale_translate_file_attach_properties($file, $options)
After****LocaleFile::createFromPath($filename, $filepath, $options['langcode'])
Before****locale_translate_delete_translation_files($projects, $langcodes)
After****\Drupal::service(LocaleFileManager::class)-&gt;deleteTranslationFiles($projects, $langcodes)
Before****locale_translation_http_check($uri)
After****\Drupal::service(LocaleFileManager::class)checkRemoteFileStatus($uri)
Before****locale_translate_file_create($filename, $filepath)
After**
new LocaleFile($filepath, basename($filepath), $hash, filemtime($filepath))
There is a new RemoteFileInfo value object to track location,  lastModified and status.

There is a new RemoteFileStatus enum to track the statuses of remote files with the following options.

- Success

- Error

- Missing

---

## 155. [Installer-specific extension list implementations are deprecated](https://www.drupal.org/node/3577846)

- **Node ID**: [3577846](https://www.drupal.org/node/3577846)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#2934063](https://www.drupal.org/node/2934063), [#2719315](https://www.drupal.org/node/2719315)

**Description:**

Drupal previously used installer-specific extension list implementations to preserve pathnames added with setPathname()  during installer container rebuilds.

That is no longer necessary. The base extension list behavior now preserves added pathnames across reset(), so the installer-specific trait and classes no**longer provide any unique behavior. Core also no longer replaces the extension.list.module and extension.list.theme services with installer-specific classes
during installation.

**What changed:**

The following installer-specific APIs are deprecated in Drupal 11.4.0 and will be removed in Drupal 13.0.0:

- Drupal\Core\Installer\ExtensionListTrait
- Drupal\Core\Installer\InstallerModuleExtensionList
- Drupal\Core\Installer\InstallerThemeExtensionList
Use these instead:

- Drupal\Core\Extension\ModuleExtensionList
- Drupal\Core\Extension\ThemeExtensionList
There is no replacement for Drupal\Core\Installer\ExtensionListTrait. The base extension list behavior now provides the same functionality.

Before**

```php
use Drupal\Core\Installer\ExtensionListTrait;
  use Drupal\Core\Installer\InstallerModuleExtensionList;
  use Drupal\Core\Installer\InstallerThemeExtensionList;

  class MyInstallerModuleExtensionList extends InstallerModuleExtensionList {
  }

  class MyInstallerThemeExtensionList extends InstallerThemeExtensionList {
  }

  class MyCustomExtensionList extends ModuleExtensionList {
    use ExtensionListTrait;
  }
```

**After:**

```php
use Drupal\Core\Extension\ModuleExtensionList;
  use Drupal\Core\Extension\ThemeExtensionList;

  class MyModuleExtensionList extends ModuleExtensionList {
  }

  class MyThemeExtensionList extends ThemeExtensionList {
  }

  class MyCustomExtensionList extends ModuleExtensionList {
  }
```

How to update your code

- If your code typehints InstallerModuleExtensionList, change it to ModuleExtensionList.
- If your code typehints InstallerThemeExtensionList, change it to ThemeExtensionList.
- If your custom extension list class uses ExtensionListTrait, remove the trait. The inherited behavior from the base extension list class is sufficient.
Notes

The deprecated installer classes remain as backward-compatibility wrappers for now, but core no longer uses them during installation.

---

## 156. [Using integer values for database connections keys and targets is deprecated](https://www.drupal.org/node/3577925)

- **Node ID**: [3577925](https://www.drupal.org/node/3577925)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3532930](https://www.drupal.org/node/3532930)

**Description:**

Using integer values for database connections keys and targets is deprecated.

In fact they were never supported, but strict typehinting introduced in [#3532930: Make Drupal\Core\Database\Database type strict and PHPStan L10 compliant](/project/drupal/issues/3532930) surfaced the bug that they might have been passed in without any error.

---

## 157. [node_access_grants has been deprecated](https://www.drupal.org/node/3578055)

- **Node ID**: [3578055](https://www.drupal.org/node/3578055)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2473041](https://www.drupal.org/node/2473041)

**Description:**

node_access_grants has been deprecated.

Use \Drupal::service(NodeGrantsHelper::class)-&gt;nodeAccessGrants($operation, $account); instead.

The following now require NodeGrantsHelper as an argument.

- NodeAccessGrantsCacheContext()

- NodeGrantDatabaseStorage()

---

## 158. [The Shortcut module is deprecated](https://www.drupal.org/node/3578141)

- **Node ID**: [3578141](https://www.drupal.org/node/3578141)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3569118](https://www.drupal.org/node/3569118)

**Description:**

The Shortcut module is deprecated and will be removed from Drupal 12.0.0.

If you want to keep using the functionality provided by Shortcut, read the [recommendations for Shortcut](https://www.drupal.org/docs/core-modules-and-themes/deprecated-and-obsolete-modules-and-themes#s-shortcut).

---

## 159. [Config storage now uses JSON for its database storage](https://www.drupal.org/node/3578326)

- **Node ID**: [3578326](https://www.drupal.org/node/3578326)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3577321](https://www.drupal.org/node/3577321)

**Description:**

The config table data column has been changed from a serialized PHP blob to a native JSON column type. Config entity queries now use SQL JSON expressions instead of loading all records into PHP.

Database drivers must support the JSON schema type. See [https://www.drupal.org/node/3389682](https://www.drupal.org/node/3389682)

---

## 160. [PHP __sleep()/__wakeup() replaced with __serialize()/__unserialize()](https://www.drupal.org/node/3578423)

- **Node ID**: [3578423](https://www.drupal.org/node/3578423)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3548971](https://www.drupal.org/node/3548971)

**Description:**

**Summary:**

Drupal core classes that implement custom serialization now use PHP's modern __serialize()/__unserialize() methods instead of the legacy __sleep()/__wakeup() pair.

**Which classes are affected:**

- DependencySerializationTrait (used by most services and plugins)

- EntityBase and ConfigEntityBase

- EntityDisplayBase

- ContainerBuilder

- Connection (database)

- Query (database)

- MemoryBackend (cache)

- StorageComparer and ConfigImporter

- Extension

- TranslatableMarkup and PluralTranslatableMarkup

- ViewExecutable

- TermStorage

**What changed:**

__sleep() returns property names and relies on PHP to serialize their values.__serialize() returns the full data array directly, giving explicit control over what gets serialized.

```php
// Before
public function __sleep(): array {
  return ['propertyA', 'propertyB'];
}
public function __wakeup(): void {
  // restore from $this->propertyA, etc.
}

// After
public function __serialize(): array {
  return [
    'propertyA' => $this->propertyA,
    'propertyB' => $this->propertyB,
  ];
}
public function __unserialize(array $data): void {
  $this->propertyA = $data['propertyA'];
  $this->propertyB = $data['propertyB'];
}
```

**Impact on contributed and custom code:**

**If you override __sleep() or __wakeup():**

Replace them with __serialize() and__unserialize(). PHP calls __serialize() in preference to__sleep() when both exist, so the old methods will no longer be invoked.

**If you use DependencySerializationTrait:**

No changes needed — the trait handles serialization automatically. If you override __sleep()/__wakeup() in a class using the trait, update those overrides to __serialize()/__unserialize().

**If you extend EntityBase or ConfigEntityBase:**

If you override __sleep(), rename to__serialize() and change the return value from property names to a key-value array. Call parent::__serialize() instead of parent::__sleep().

**Why:**

- __serialize()/__unserialize() is the recommended PHP serialization API since PHP 7.4

- Avoids side effects — __sleep() often requires storing temporary state in instance properties just to pass data to__wakeup()

- Clearer data flow — serialized data is an explicit array, not implicit property access

- Better performance — no separate property-name resolution step

- Setting readonly class properties in __wakeup() causes errors, which do not occur with__unserialize()

---

## 161. [Test methods consolidated in EntityResourceTestBase](https://www.drupal.org/node/3578858)

- **Node ID**: [3578858](https://www.drupal.org/node/3578858)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3575888](https://www.drupal.org/node/3575888)

**Description:**

Public methods on EntityResourceTestBase have been moved to protected methods on the class:

```php
::testGet()
 ::testPost()
 ::testPatch()
 ::testDelete()
```

```php
::doTestGet()
 ::doTestPost()
 ::doTestPatch()
 ::doTestDelete()
```

These protected methods are now called by a single public ::testCrud() method.
Contrib tests that subclass ResourceTestBase should not need to make any changes unless they override one of these specific methods.

---

## 162. [AccessResult::allowedIf() and AccessResult::forbiddenIf() now accept a neutral reason](https://www.drupal.org/node/3579037)

- **Node ID**: [3579037](https://www.drupal.org/node/3579037)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3566611](https://www.drupal.org/node/3566611)

**Description:**

Before it was possible to add a reason to a neutral access result when using AccessResult::neutral(), but not when using AccessResult::allowedIf() or AccessResult::forbiddenIf().

Now these methods accept a neutral reason argument:

```php
AccessResult::allowedIf($condition, 'This is neutral because...');
AccessResult::forbiddenIf($condition, 'This is forbidden because...', 'This is neutral because...');
```

---

## 163. [\Drupal\Core\Recipe\RecipeRunner::installModule() is deprecated](https://www.drupal.org/node/3579527)

- **Node ID**: [3579527](https://www.drupal.org/node/3579527)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3498026](https://www.drupal.org/node/3498026)

**Description:**

Drupal’s recipe system has been updated to support installing multiple modules using batching with config isolation. This significantly improves performance.

The RecipeRunner now supports installing modules in chunks rather than one by one. The batch size is configurable via a new setting, core.multi_module_install_batch_size (which defaults to 20).

\Drupal\Core\Recipe\RecipeRunner::installModule(string $module, ...) has been deprecated in Drupal 11.4.0 and will be removed in Drupal 13.0.0.

Developers and extension authors should use \Drupal\Core\Recipe\RecipeRunner::installModules() instead to leverage multi-module batch processing.

---

## 164. [Page cache request and response policies now use tagged iterators instead of service collectors](https://www.drupal.org/node/3579668)

- **Node ID**: [3579668](https://www.drupal.org/node/3579668)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3432818](https://www.drupal.org/node/3432818)

**Description:**

The page cache request and response policy chain classes (\Drupal\Core\PageCache\ChainRequestPolicy and \Drupal\Core\PageCache\ChainResponsePolicy) now accept their policy rules via tagged iterators injected through the constructor.

If your code type hints \Drupal\Core\PageCache\ChainRequestPolicyInterface or \Drupal\Core\PageCache\ChainResponsePolicyInterface, these interfaces are deprecated. Change type hints to \Drupal\Core\PageCache\RequestPolicyInterface or \Drupal\Core\PageCache\ResponsePolicyInterface respectively.

If your code extends \Drupal\Core\PageCache\DefaultRequestPolicy or \Drupal\dynamic_page_cache\PageCache\RequestPolicy\DefaultRequestPolicy, these classes are deprecated. Use \Drupal\Core\PageCache\ChainRequestPolicy with tagged services directly instead.

The CommandLineOrUnsafeMethod and NoSessionOpen policies are now registered as individual tagged services in core.services.yml rather than being hardcoded inside DefaultRequestPolicy.

---

## 165. [New Exception status code cache context](https://www.drupal.org/node/3580452)

- **Node ID**: [3580452](https://www.drupal.org/node/3580452)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3516173](https://www.drupal.org/node/3516173)

**Description:**

Drupal core includes a block visibility condition to control whether blocks are shown on 404 or 403 pages or not.

This visibility condition added the url.path cache context, which lead to low cache hit rates for the block render cache on 404 and 403 pages.

A new exception_status_code cache context has been added, which returns the status code when there is an exception, or the string '0' when there is no request exception. This is used in the block condition.

---

## 166. [Using jQuery sizzle selectors has been deprecated](https://www.drupal.org/node/3580619)

- **Node ID**: [3580619](https://www.drupal.org/node/3580619)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3568777](https://www.drupal.org/node/3568777)

**Description:**

In Drupal 11.4.0, use of jQuery-only selector extensions (Sizzle selectors) is deprecated in selector strings handled by core AJAX command flows.

Selectors are validated as CSS selectors. If a selector is not valid for document.querySelector(), Drupal triggers a deprecation warning and still falls back to jQuery selector handling for backward compatibility in Drupal 11.

This compatibility layer is removed in Drupal 12. Contributed and custom code should migrate to standard CSS selectors.

[https://api.jquery.com/category/selectors/jquery-selector-extensions/](https://api.jquery.com/category/selectors/jquery-selector-extensions/)**[https://learn.jquery.com/performance/optimize-selectors/](https://learn.jquery.com/performance/optimize-selectors/)

**Examples:**

Common jQuery/Sizzle selectors and CSS-compatible replacements:

Deprecated**: $('.form-wrapper :checkbox')**Replacement**: $('.form-wrapper input[type="checkbox"]')
**Deprecated**: $('input:checkbox, input:radio')**Replacement**: $('input[type="checkbox"], input[type="radio"]')
**Deprecated**: $('.filters :input:not(:checked)')**Replacement**: $('.filters input:not(:checked)')
**Deprecated**: $('tr:eq(0)')**Replacement**: $('tr').eq(0)
**Deprecated**: $('li:lt(3)')**Replacement**: $('li').slice(0, 3)
**Deprecated**: $('a:contains("Download")')**Replacement**:

```php
$('a').filter(function () {
  return $(this).text().includes('Download');
})
```

**Deprecated**: $('tbody tr:visible')**Replacement**:

```php
$('tbody tr').filter(function () {
  return this.offsetParent !== null;
})
```

**What this means for developers:**

- Do not pass jQuery-only pseudo-selectors in selector strings that are expected to be CSS-compatible.
- Prefer standard CSS selectors and then apply jQuery filtering or methods as a second step when needed.
- Update custom AJAX command selectors and any dynamically generated selectors accordingly.

---

## 167. [User account cancellation is handled by a service. Cancellation methods are now plugins](https://www.drupal.org/node/3580745)

- **Node ID**: [3580745](https://www.drupal.org/node/3580745)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3580682](https://www.drupal.org/node/3580682)

**Description:**

The user cancellation feature has been refactored:

**Service to cancel user accounts:**

A new service, Drupal\user\AccountCancellation, was introduced to replace current procedural code related ti user account cancellation.

**Cancel an account:**

**Before:**

```php
$edit = ['user_cancel_notify' => TRUE];
user_cancel($edit, 123, 'user_cancel_block');
```

**After:**

```php
use Drupal\user\AccountCancellation;
use Drupal\user\Entity\User;
...
$context = ['user_cancel_notify' => TRUE];
$account = User::load(123);
// Inject service where possible.
\Drupal::service(AccountCancellation::class)->cancel($account, 'user_cancel_block', $context);
```

**Define custom cancellation methods:**

**Before:**

```php
function my_module_user_cancel_methods_alter(&$methods) {
  $methods['my_module_zero_out'] = [
    'title' => t('Delete the account and remove all content.'),
    'description' => t('All your content will be replaced by empty strings.'),
    // Access should be used for administrative methods only.
    'access' => $account->hasPermission('access zero-out account cancellation method'),
  ];
}
```

**After:**

Create a new plugin in my_module, under src/Plugin/user/CancelMethod

```php
#[AccountCancelMethod(
  id: 'zero_out',
  label: new TranslatableMarkup('Delete the account and remove all content.'),
  description: new TranslatableMarkup('All your content will be replaced by empty strings.'),
)]
class ZeroOut extends AccountCancelMethodPluginBase {
  public function access(AccountInterface $account): bool {
    return $account->hasPermission('access zero-out account cancellation method');
  }
  public function cancel(UserInterface $account, array $context = []): void {
    \Drupal::logger('my_module')->notice('We will miss you!');
  }
}
```

**Get the cancellation method form element:**

**Before:**

```php
$methods = user_cancel_methods();
```

**After:**

```php
use Drupal\user\AccountCancellation;
...
// Inject service where possible.
$methods = \Drupal::service(AccountCancellation::class)->getMethodsFormElement();
```

**Other deprecations:**

**Procedural functions::**

Function
Replacement

_user_cancel()
\Drupal\user\AccountCancellation::cancelAccount()

_user_cancel_session_regenerate()
\Drupal\user\AccountCancellation::regenerateSession()

**Constructor parameter additions:**

Class
Service
Position

Drupal\jsonapi\Controller\EntityResource
Drupal\user\AccountCancellation
12

Drupal\user\Controller\UserController
Drupal\user\AccountCancellation
6

Drupal\user\Form\UserCancelForm
Drupal\user\AccountCancellation
3

Drupal\user\Form\UserMultipleCancelConfirm
Drupal\user\AccountCancellation
3

Drupal\user\AccountSettingsForm
Drupal\user\AccountCancellation
3

Drupal\user\AccountSettingsForm
Drupal\user\AccountCancellation
3

---

## 168. [Passing entity storage to constructor was deprecated for several classes](https://www.drupal.org/node/3581019)

- **Node ID**: [3581019](https://www.drupal.org/node/3581019)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3581014](https://www.drupal.org/node/3581014)

**Description:**

TBD

---

## 169. [user_pass_rehash(), user_cancel_url(), user_mail_tokens(), and user_pass_reset_url() are deprecated](https://www.drupal.org/node/3581062)

- **Node ID**: [3581062](https://www.drupal.org/node/3581062)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3581056](https://www.drupal.org/node/3581056)

**Description:**

Functions related to one time authentication are deprecated in Drupal 11.4.0 and removed from Drupal 13.0.0. Use the methods on \Drupal\user\OneTimeAuthentication instead.

Deprecated
Replacement

user_pass_rehash(): string
\Drupal\user\OneTimeAuthentication::generateHmac(): \Drupal\Core\Url

user_cancel_url(): string
\Drupal\user\OneTimeAuthentication::generateCancelConfirmUrl(): \Drupal\Core\Url

user_pass_reset_url(): string
\Drupal\user\OneTimeAuthentication::generateOneTimeLoginUrl(): \Drupal\Core\Url

user_mail_tokens(): string
\Drupal\user\OneTimeAuthentication::tokens(): \Drupal\Core\Url

The following protected helper functions are deprecated in Drupal 11.4.0 and removed from Drupal 12.0.0.

Deprecated
Replacement

\Drupal\user\Controller\UserController::validatePathParameters()
\Drupal\user\OneTimeAuthentication::verifyHmac()

\Drupal\Core\Command\ServerCommand::getOneTimeLoginUrl()
\-
**Example 1:**

**Before:**

```php
$href = user_cancel_url($account);
```

**After:**

```php
$oneTimeAuthentication = \Drupal->service(OneTimeAuthentication::class);
$href = $oneTimeAuthentication->generateCancelConfirmUrl($account)->toString();
```

**Example 2:**

**Before:**

```php
#[Hook('mail')]
  public function mail($key, &$message, $params): void {
    [...]
    $options = [
      'langcode' => $langcode,
      'callback' => 'user_mail_tokens',
      'clear' => TRUE,
    ];
    $message['body'][] = $token_service->replacePlain($body, $context, $options);
    [...]
  }
```

**After:**

```php
#[Hook('mail')]
  public function mail($key, &$message, $params): void {
    [...]
    $options = [
      'langcode' => $langcode,
      'callback' => \Drupal::service(OneTimeAuthentication::class)->tokens(...),
      'clear' => TRUE,
    ];
    $message['body'][] = $token_service->replacePlain($body, $context, $options);
    [...]
  }
```

---

## 170. [Class Variance Authority (CVA) support added to Twig](https://www.drupal.org/node/3581193)

- **Node ID**: [3581193](https://www.drupal.org/node/3581193)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3559156](https://www.drupal.org/node/3559156)

**Description:**

Drupal core has added the twig/html-extra package as a dependency and now exposes the html_cva() Twig function, which implements the Class Variance Authority (CVA) pattern for managing conditional CSS class usage in templates. See [https://twig.symfony.com/doc/3.x/functions/html_cva.html](https://twig.symfony.com/doc/3.x/functions/html_cva.html) for documentation on this function and how to use it.

---

## 171. [User and Media Document and Image create links removed from navigation](https://www.drupal.org/node/3581445)

- **Node ID**: [3581445](https://www.drupal.org/node/3581445)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3578196](https://www.drupal.org/node/3578196)

**Description:**

The Navigation module provides a 'Content' menu that includes links to create entities of certain types and bundles. These links would appear under the "Create" link in the navigation sidebar. This previously included hardcoded links to create User entities, as well as Image and Document Media entities if the Media module is installed. These hardcoded links have been removed.

Example:**Before****
After**

The links can be manually added back to the Content menu for any site with users who want to continue to use them.
Users who have access to edit menus can go to /admin/structure/menu/manage/content and add them below the "Create" link:

---

## 172. [user_cookie_save() and user_cookie_delete() are deprecated](https://www.drupal.org/node/3581570)

- **Node ID**: [3581570](https://www.drupal.org/node/3581570)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3581569](https://www.drupal.org/node/3581569)

**Description:**

The functions user_cookie_save() and user_cookie_delete() are deprecated. Use the setCookie() and clearCookie() methods on the Symfony Response::headers instead.

**Example:**

**Before:**

```php
class MyController extends ControllerBase {

  public function compactPage($mode) {
    user_cookie_save(['admin_compact_mode' => ($mode == 'on')]);
    return $this->redirect('');
  }

}
```

**After:**

```php
class MyController extends ControllerBase {

  public function compactPage($mode) {
    $response = $this->redirect('');
    if ($mode === 'on') {
      $response->headers->setCookie(new Cookie('Drupal.visitor.admin_compact_mode', '1', $this->time->getRequestTime() + 31536000));
    }
    else {
      $response->headers->clearCookie('Drupal.visitor.admin_compact_mode');
    }
    return $response;
  }

}
```

---

## 173. [Error reporting in test child sites changed from HTTP headers to log files and _drupal_error_header() is deprecated](https://www.drupal.org/node/3581815)

- **Node ID**: [3581815](https://www.drupal.org/node/3581815)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3580572](https://www.drupal.org/node/3580572)

**Description:**

When a child site (the Drupal site running under test) encounters an error or deprecation, it previously relayed that information to the test runner by injecting custom X-Drupal-Assertion-* HTTP response headers. This mechanism fails sometimes ebcause headers cannot be sent after output has started.

The header-based approach has been replaced with a file-based approach. Each child site now writes serialized error data to a per-process log file inside a**logs/ subdirectory of the test site path. After each request the test runner reads those files, re-triggers deprecations via trigger_error() so PHPUnit can collect them, and throws exceptions for all other error types.

**New API:**

- \Drupal\Core\Test\TestErrorLogger** – new final class that handles writing errors to the log file (writeError()), reading them back, and re-triggering them (reTriggerErrors()).

- **\Drupal\Core\Test\Exception\TestErrorLoggerIgnoreException** – a marker exception that TestErrorLogger will silently drop, useful when a test intentionally triggers an exception and wants to inspect the rendered HTML output without the error being re-thrown by the runner.

- **\Drupal\Core\Test\TestDatabase::getLogDirectory()** – new method returning the path to the test's logs/ directory. The directory is guaranteed to exist and be writable.

- **\Drupal\Core\Test\TestSetupTrait::prepareDatabasePrefix()** – now returns the TestDatabase object instead of void.

**Deprecated:**

- _drupal_error_header() is deprecated in drupal:11.4.0 and will be removed in drupal:13.0.0. Replace calls with \Drupal\Core\Test\TestErrorLogger::writeError().

---

## 174. [Password hashing is configurable using kernel parameters](https://www.drupal.org/node/3581980)

- **Node ID**: [3581980](https://www.drupal.org/node/3581980)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3581966](https://www.drupal.org/node/3581966)

**Description:**

The password hashing algorithm and options can be changed using kernel parameters. These parameters are passed to [password_hash()](https://php.net/password_hash) whenever a password is created or changed. The algorithm parameter defaults to null and the options parameter to [] (empty array).

Developers may change the algorithm used via custom services.yml file loaded via settings.php.

**Custom services.yml example:**

If you have an existing services.yml file in your sites folder (e.g. sites/default/files/services.yml),
you can simply add two new parameters. In most cases you won't need to add password.options as the default options will suffice.

```php
parameters:
  # Can be argon2i, argon2id or 2y
  password.algorithm: argon2id # 👈️ Parameter 1
  # See https://www.php.net/password_hash
  password.options: [] # 👈️ Parameter 2 - optional
```

If you don't have an existing services.yml file, you can create one and load it by adding this to your settings.php

```php
// Add to your settings.php - use the path of the file you created.
$settings['container_yamls'][] = DRUPAL_ROOT . '/sites/default/services.yml';
```

**Drupal 12:**

Default password hashing algorithm is [argon2id in Drupal 12](https://www.drupal.org/node/3530196/).

**Forwards compatibility layer:**

Site owners wishing to take advantage of this functionality before Drupal 11.4 is released can install the 3.0.0 series of the [PHP Password](/project/php_password) contributed module. It provides a forward compatibility layer for this functionality.

---

## 175. [404 responses are now a CacheableNotFoundHttpException (Router::matchRequest() throws CacheableResourceNotFoundException)](https://www.drupal.org/node/3581985)

- **Node ID**: [3581985](https://www.drupal.org/node/3581985)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3580545](https://www.drupal.org/node/3580545)

**Description:**

Router::matchRequest() no throws a (newly introduced) CacheableResourceNotFoundException instead of a ResourceNotFoundException if now route can be found for a given request.

This will result in a CacheableNotFoundHttpException being returned for a 404. This does not change the *actual* caching because 404 responses are outside of the scope of Dynamic Page Cache and Page Cache already cached 404s previously, but it does result in additional cacheability headers being returned if the http.response.debug_cacheability_headers service parameter is enabled. The headers will generally be

Name
Value

```text
X-Drupal-Cache-Tags
```

```text
4xx-response http_response
```

```text
X-Drupal-Cache-Contexts
```

```text

```

```text
X-Drupal-Cache-Max-Age
```

```text
-1 (Permanent)
```

This is preparing for [#2352009: Bubbling of elements' max-age to the page's headers and the page cache](/project/drupal/issues/2352009), since that change would result in no longer caching 404 responses in page cache, combined with this change, there behavior in regards to those will remain unchanged.

---

## 176. [user_form_process_password_confirm() is deprecated](https://www.drupal.org/node/3582107)

- **Node ID**: [3582107](https://www.drupal.org/node/3582107)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3582106](https://www.drupal.org/node/3582106)

**Description:**

user_form_process_password_confirm() is deprecated in and is removed from Drupal 13. Use UserThemeHooks::processPasswordConfirm() instead.

**Example:**

**Before:**

```php
#[Hook('element_info_alter')]
  public function elementInfoAlter(array &$types): void {
    if (isset($types['password_confirm'])) {
      $types['password_confirm']['#process'][] = 'user_form_process_password_confirm';
    }
  }
```

**After:**

```php
#[Hook('element_info_alter')]
  public function elementInfoAlter(array &$types): void {
    if (isset($types['password_confirm'])) {
      $types['password_confirm']['#process'][] = UserHooks::class . ':processPasswordConfirm';
    }
  }
```

---

## 177. [Inline links in help topics are no longer rendered as absolute](https://www.drupal.org/node/3582394)

- **Node ID**: [3582394](https://www.drupal.org/node/3582394)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3582386](https://www.drupal.org/node/3582386)

**Description:**

Links rendered with either of the 'help_route_link' or 'help_topic_link' twig functions are no longer rendered as absolute by default. This behaviour can be customised by passing in $options['absolute'] = TRUE, although this is unlikely to be necessary.

---

## 178. [Uninstalling themes in the UI now have a confirmation step](https://www.drupal.org/node/3582449)

- **Node ID**: [3582449](https://www.drupal.org/node/3582449)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3096170](https://www.drupal.org/node/3096170)

**Description:**

When uninstalling a theme in the UI a confirmation form will now be presented outlining the configuration that will be removed.

This grants the ability to cancel the uninstall.

---

## 179. [The persist service tag is removed](https://www.drupal.org/node/3584017)

- **Node ID**: [3584017](https://www.drupal.org/node/3584017)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#2368263](https://www.drupal.org/node/2368263)

**Description:**

The persist service tag is removed. Instead the request stack must be manually set up after a container rebuild.

---

## 180. [views_invalidate_cache has been deprecated.](https://www.drupal.org/node/3584050)

- **Node ID**: [3584050](https://www.drupal.org/node/3584050)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#941970](https://www.drupal.org/node/941970)

**Description:**

views_invalidate_cache() has been deprecated.

It has been split into two methods Views::invalidateCache() and Views::routeRebuildNeeded()

The logic has been moved to each plugin's postSaveView

The configuration changes are now checked to determine which action to take.
Critically route rebuilds should only happen when actually necessary.

---

## 181. [X-Drupal-Dynamic-Cache response header updated for 4xx and 5xx responses](https://www.drupal.org/node/3584640)

- **Node ID**: [3584640](https://www.drupal.org/node/3584640)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3451483](https://www.drupal.org/node/3451483)

**Description:**

Previously the value of the X-Drupal-Dynamic-Cache response header for 4xx and 5xx responses was misinforming developers: if the 4xx or 5xx response was generated early enough, the response would state X-Drupal-Dynamic-Cache: UNCACHEABLE (poor cacheability), even though cacheability played no role. It was uncacheable because the response was generated so early.

This has now been fixed so that 4xx and 5xx responses always return UNCACHEABLE (XXX) as the value of the X-Drupal-Dynamic-Cache response header where XXX is the actual response status code (so in particular UNCACHEABLE (403) or UNACHEABLE (404), for example).

For "regular" 4xx requests (i.e. for GET requests with HTML content type), Drupal internally performs a sub-request to return the (in the case of a 404, for example) "not found" page. In case that happens, the cache status of that sub-request is also returned as part of the value of the X-Drupal-Dynamic-Cache response header. The value is then UNCACHEABLE (403, sub-request: MISS) or UNCACHEABLE (404, sub-request: HIT), for example.

---

## 182. [dr - Drupal CLI capable of running commands from modules](https://www.drupal.org/node/3584928)

- **Node ID**: [3584928](https://www.drupal.org/node/3584928)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3453474](https://www.drupal.org/node/3453474), [#3583795](https://www.drupal.org/node/3583795)

**Description:**

The command-line interface (CLI) in core/scripts/drupal had limited use for core-provided commands only. The CLI and all existing commands are refactored to be discoverable by vendor/bin/dr (implemented in core/scripts/dr).

**Summary of changes:**

- Commands created by modules can be discovered by using the AsCommand Attribute.

- The entry point for the drupal CLI, WEBROOT/core/scripts/drupal, is deprecated and replaced with WEBROOT/core/scripts/dr, which is installed into composer’s bin-dir as vendor/bin/dr. It is preferred to run the CLI from vendor/bin/dr or locally in DDev ddev exec dr.

- Drupal\Core\Command\BootableCommandTrait is deprecated and no longer necessary.

- Drupal\Core\Command\CacheRebuildCommand is implemented to serve as an example and can be used as vendor/bin/dr cache:rebuild to rebuild caches.

- Drupal\system\Command\StatusCommand and Drupal\system\Command\CronCommand are implemented to serve as examples for module-provided commands.

- Commands should use the Symfony Command return types Command::SUCCESS, Command::ERROR, etc...

- The dr CLI will log output to the logger.console / Drupal\Core\Command\DrupalConsoleLogger logger.

The dr CLI and commands should be considered @internal and experimental as more commands are added to Drupal Core. It is recommended to join #cli-in-core [Drupal Slack](https://drupal.org/slack) channel or follow discussions in issues under [[meta] CLI in Core community initiative](https://www.drupal.org/node/3582246) to discuss any contributed or custom module requirements.

**Providing commands:**

Modules implementing commands should use their module name as the namespace when creating commands substituting hyphens for underscores in the namespace and command name e.g. my_module becomes my-module:my-command. Commands without a namespace, core, and core module namespaces are reserved for Drupal core, and using them may result in conflicts in the future.

**Before:**

```php
final class RecipeInfoCommand extends Command {

  use BootableCommandTrait;

  public function __construct($class_loader) {
    parent::__construct('recipe:info');
    $this->classLoader = $class_loader;
  }

  protected function execute() {
    $this->boot();
    // ...
  }

  // ...

}
```

**After:**

- Use the AsCommand class attribute for discovery. Once Drupal core adopts Symfony 8.1, the AsCommand attribute will be available for methods as well allowing multiple commands on a class.

- Use the Autowire attribute for service injection.

```php
use Composer\Autoload\ClassLoader;
use Symfony\Component\Console\Attribute\AsCommand;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Style\SymfonyStyle;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

#[AsCommand(
  name: 'recipe:info',
  description: 'Shows information about a recipe.',
)]
final class RecipeInfoCommand extends Command {

  public function __construct(
    #[Autowire(service: 'class_loader')]
    protected ClassLoader $class_loader,
  ) {
    parent::__construct();
  }

  protected function execute(InputInterface $input, OutputInterface $output): int {
    $io = new SymfonyStyle($input, $output);
    // ...
  }

  // ...

}
```

**dr CLI usage:**

```php
% ./vendor/bin/dr
Drupal 11.4.0

Usage:
  command [options] [arguments]

Options:
  -h, --help            Display help for the given command. When no command is given display help for the list command
      --silent          Do not output any message
  -q, --quiet           Only errors are displayed. All other output is suppressed
  -V, --version         Display this application version
      --ansi|--no-ansi  Force (or disable --no-ansi) ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --url=URL         A base URL (e.g. example.com). Used for building links, selecting a multi-site, etc.
  -e, --env=ENV         The Environment name. [default: "prod"]
      --no-debug        Switches off debug mode.
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands:
  completion      Dump the shell completion script
  generate-theme  Generates a new theme based on latest default markup.
  help            Display help for a command
  install         Install Drupal using an install profile or recipe.
  list            List commands
  quick-start     Installs a Drupal site and starts a web server for local testing or development.
  server          Starts up a webserver for a site.
 cache
  cache:rebuild   [cr|rebuild] Rebuild all caches.
 content
  content:export  Exports content entities in YAML format.
 recipe
  recipe:apply    [recipe] Applies a recipe to a site.
  recipe:info     Shows information about a recipe.
 system
  system:cron     [cron|core:cron] Runs cron implementations.
  system:status   [status] Show system status.
```

**Porting guide for Drush command authors:**

[A full porting guide is available at for Drush command authors](https://www.drupal.org/docs/develop/drupal-apis/command-line-interface-cli-api/drush-command-porting-guide-to-the-new-dr-drupal-core-cli ).

---

## 183. [Entity query methods no longer implicitly support passing different query objects](https://www.drupal.org/node/3585318)

- **Node ID**: [3585318](https://www.drupal.org/node/3585318)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2875033](https://www.drupal.org/node/2875033)

**Description:**

Drupal\Core\Entity\Query\Sql\Query::getTables() no longer takes an $sql_query parameter and passing it in will trigger a deprecation in 11.4.0 and have no effect in 13.0.0. The query object is determined from the entity query object itself, so the argument can be removed.

Additionally, Drupal\Core\Entity\Query\Sql\Query::condition() will no longer accept a Condition object where that object was generated from a different entity query object. This was never officially supported, but will be explicitly disallowed from Drupal 13.0.0

---

## 184. [String formatter can now also link to an entity's edit form](https://www.drupal.org/node/3585330)

- **Node ID**: [3585330](https://www.drupal.org/node/3585330)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3544339](https://www.drupal.org/node/3544339)

**Description:**

From 11.4.0 the string formatter (available on most string fields) can now also link to an entity's edit form.

If the entity has an edit form and a canonical link, site builders can choose which link relation to link to

If you alter this form and you are not seeing your options, ensure you add

```php
$form['link_rel']['#access'] = count($form['link_rel']['#options']) > 1;
```

after setting them.

---

## 185. [StreamWrapperManager::register() is deprecated](https://www.drupal.org/node/3585389)

- **Node ID**: [3585389](https://www.drupal.org/node/3585389)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3583911](https://www.drupal.org/node/3583911)

**Description:**

StreamWrapperManager::register() previously was called from a number of places throughout core in order to register tagged stream wrapper services.

This registration now happens in the StreamWrapperManager constructor so the register() method is obsolete. Any calls to the register() method can be removed.

---

## 186. [Library definitions now support a fonts key for preloading](https://www.drupal.org/node/3585574)

- **Node ID**: [3585574](https://www.drupal.org/node/3585574)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3366561](https://www.drupal.org/node/3366561)

**Description:**

Library definitions now support a fonts key, for preloading of fonts.

Previously Drupal did not provide an API for font preloading, although there were patterns such as adding the links manually to the html header template.

Fonts to be preloaded can now be declared at the same level as CSS and JavaScript in the library definition, and a &lt;link rel="preload" element will be added to the HTML header programmatically.

Note that setting preload: false in a definition is the same as not including a font in the definition, it has no effect.

```php
global-styling:
  version: VERSION
  fonts:
    fonts/metropolis/Metropolis-Regular.woff2:
      preload: true
```

Fonts should be preloaded when they're very likely to block rendering above the fold. See [https://web.dev/learn/performance/optimize-web-fonts](https://web.dev/learn/performance/optimize-web-fonts) for more details

---

## 187. [DrupalKernel container storage API changes](https://www.drupal.org/node/3585642)

- **Node ID**: [3585642](https://www.drupal.org/node/3585642)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3583505](https://www.drupal.org/node/3583505)

**Description:**

Container loading, dumping, and invalidation have been extracted from DrupalKernel into the new \Drupal\Core\DependencyInjection\ContainerStorageInterface. Several DrupalKernel methods, properties, and the $phpArrayDumperClass property are deprecated. New classes and an updated PhpContainer base class are introduced.

**New ContainerStorageInterface and implementations:**

\Drupal\Core\DependencyInjection\ContainerStorageInterface is the new abstraction for container persistence. Three implementations are provided:

- ContainerStorage\CacheContainerStorage — stores the container as a serialised PHP array in cache.container. This is the legacy behaviour, now encapsulated in its own class.

- ContainerStorage\PhpDumperContainerStorage — compiles the container via Symfony's PhpDumper into a uniquely-named PHP class stored in PhpStorage. The class name is derived from an xxh128 hash of the generated code so that concurrent requests during a rebuild never load a stale class.

- ContainerStorage\NullContainerStorage — always returns NULL from load() and returns the ContainerBuilder unchanged from dump(). Used by InstallerKernel, UpdateKernel, and TestKernel where persistent storage is not wanted.

**New PhpContainer base class:**

\Drupal\Core\DependencyInjection\PhpContainer is the new base class for containers compiled by PhpDumperContainerStorage.

**Deprecated DrupalKernel members:**

The following are deprecated in Drupal 11.4.0 and removed in Drupal 13.0.0:

DrupalKernel::$phpArrayDumperClass
Previously used to swap the dumper class used when serialising the container to the cache bin. Override the container_storage.cache service in the bootstrap container definition, or pass a custom $dumperClass as the third constructor argument of CacheContainerStorage, instead.
DrupalKernel::getCachedContainerDefinition()
Previously returned the raw PHP array definition from the cache bin. Use ContainerStorageInterface::load() instead.
DrupalKernel::getContainerCacheKey()
Previously returned the full cache key string (prefix + suffix). Use the new protected DrupalKernel::getContainerCacheKeySuffix() instead. The full key with its prefix is now an internal detail of each ContainerStorageInterface implementation.
DrupalKernel::cacheDrupalContainer()
Previously wrote the container definition directly to the cache bin. Use ContainerStorageInterface::dump() instead.

---

## 188. [Use a container dumped to PHP](https://www.drupal.org/node/3585820)

- **Node ID**: [3585820](https://www.drupal.org/node/3585820)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3583505](https://www.drupal.org/node/3583505)

**Description:**

Drupal's service container can now be compiled and stored as a PHP class file using Symfony's PhpDumper. The resulting class is stored via PhpStorage and benefits from OPcache, which can reduce container build time on warm requests. New Drupal installations default to this mode. Existing sites that upgrade continue to use the legacy PHP array-based container for backward compatibility.

**What to do:**

To switch to the compiled PHP container, add the following line to your settings.php:

```php
$settings['container_base_class'] = \Drupal\Core\DependencyInjection\PhpContainer::class;
```

New Drupal installations include this line in default.settings.php by default. Sites upgrading from an earlier version will not have it and continue to use the PHP array-based container unless they add it manually.
The container_base_class setting controls which container implementation is used at runtime:

- **\Drupal\Core\DependencyInjection\PhpContainer** — uses Symfony's PhpDumper to compile the container into a PHP class file stored in PhpStorage (typically sites/default/files/php/service_container/). The file is included on each request and benefits from OPcache.

- **\Drupal\Core\DependencyInjection\Container** (or unset) — legacy mode using the PHP array-based container serialised to the cache.container cache bin. This remains the default for sites that do not set container_base_class in order to preserve backward compatibility.

**Notes:**

- The compiled PHP container file is stored in sites/default/files/php/service_container/ (or wherever your php PhpStorage directory is configured).

- When clearing caches (drush cr or the UI cache rebuild), the old container file is deleted if the container changes..

---

## 189. [New services added to bootstrap container](https://www.drupal.org/node/3585822)

- **Node ID**: [3585822](https://www.drupal.org/node/3585822)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3583505](https://www.drupal.org/node/3583505)

**Description:**

Container persistence is now managed through a new ContainerStorageInterface abstraction. The active implementation is provided as a new service in the bootstrap container and selected automatically based on the container_base_class setting. Sites that previously overrode $settings['bootstrap_container_definition'] need to ensure their definition includes the new storage services, or override the container_storage service to customise how the container is stored.

**New services in the default bootstrap container:**

The following services are now part of DrupalKernel::$defaultBootstrapContainerDefinition:

settings
The \Drupal\Core\Site\Settings singleton. Other bootstrap services use this to read settings without depending on the not-yet-built main container.
container_storage
The active \Drupal\Core\DependencyInjection\ContainerStorageInterface implementation selected by \Drupal\Core\DependencyInjection\ContainerStorage\ContainerStorageFactory::create().
container_storage.cache
\Drupal\Core\DependencyInjection\ContainerStorage\CacheContainerStorage — the legacy implementation that serialises the container definition as a PHP array using OptimizedPhpArrayDumper and stores it in the cache.container cache bin.
container_storage.php_dumper
\Drupal\Core\DependencyInjection\ContainerStorage\PhpDumperContainerStorage — the new implementation that uses Symfony's PhpDumper to compile the container into a PHP class stored via php_storage.service_container.
php_storage.service_container
A \Drupal\Component\PhpStorage\PhpStorageInterface instance for the service_container storage bin. Provided via PhpStorageFactory::get('service_container').

**Overriding the container storage:**

To replace the container storage entirely — for example, to use a custom persistence mechanism — override the container_storage service in $settings['bootstrap_container_definition'] in settings.php:

```php
$settings['bootstrap_container_definition']['services']['container_storage'] = [
    'class' => \MyModule\MyCustomContainerStorage::class,
    // Add any constructor arguments your class requires:
    // 'arguments' => ['@some_bootstrap_service'],
  ];
```

Your class must implement \Drupal\Core\DependencyInjection\ContainerStorageInterface.

**Backward compatibility for existing bootstrap_container_definition overrides:**

If your $settings['bootstrap_container_definition'] does not declare any of the new storage services, Drupal will merge the missing services in automatically.

---

## 190. [mysqli driver connections can be configured to skip usage of prepared statements](https://www.drupal.org/node/3586360)

- **Node ID**: [3586360](https://www.drupal.org/node/3586360)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3571186](https://www.drupal.org/node/3571186), [#3384763](https://www.drupal.org/node/3384763)

**Description:**

The experimental mysqli driver can be instructed to execute queries without making use of prepared statements. In such case, the queries will be executed directly from a properly escaped SQL statement string.

This feature is meant to be used in the context of enabling async queries which is under development in [#3384763: Async database query + fiber support](/project/drupal/issues/3384763). Normal connections do not need to be configured in this sense.

The way to enable the feature is to pass in the connection options array of the database connection the key=&gt;value 'use_prepared_statements' =&gt; FALSE. It can be done in the setttings.php configuration file or programmatically.

---

## 191. [SqlContentEntityStorage::loadFromSharedTables() is deprecated](https://www.drupal.org/node/3586362)

- **Node ID**: [3586362](https://www.drupal.org/node/3586362)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3561960](https://www.drupal.org/node/3561960)

**Description:**

The protected method SqlContentEntityStorage::loadFromSharedTables() is deprecated without replacement. Its logic has been consolidated into SqlContentEntityStorage::loadFromDedicatedTables()

Contributed or custom code that is subclassing SqlContentEntityStorage will need reviewing to see whether it needs to be updated, however it is very unusual to customise this logic.

---

## 192. [\Drupal\Component\FileSystem\FileSystem::getOsTemporaryDirectory() checks the directory returned by sys_get_temp_dir() before /tmp and windows specific directories](https://www.drupal.org/node/3586712)

- **Node ID**: [3586712](https://www.drupal.org/node/3586712)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3586645](https://www.drupal.org/node/3586645)

**Description:**

\Drupal\Component\FileSystem\FileSystem::getOsTemporaryDirectory() now checks the directory returned by sys_get_temp_dir() before '/tmp'. This can result in temporary files being created in a different location.

---

## 193. [Igbinary is now the default object-aware serializer when the extension is available](https://www.drupal.org/node/3586838)

- **Node ID**: [3586838](https://www.drupal.org/node/3586838)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3014514](https://www.drupal.org/node/3014514)

**Description:**

Drupal core now ships an additional serializer, Drupal\Component\Serialization\IgbinarySerialize, registered as the serialization.igbinary service. This service is now bound to the Drupal\Component\Serialization\ObjectAwareSerializationInterface alias, replacing the previous binding to serialization.phpserialize.

When the [igbinary PHP extension](https://www.php.net/manual/en/book.igbinary.php) is installed, payloads written by the database cache backend (cache.backend.database) and the expirable database key/value store (keyvalue.expirable.database) are encoded with igbinary, which is faster to decode and produces smaller payloads than PHP's native serialize(). When the extension is not installed, IgbinarySerialize transparently falls back to serialize()/unserialize(), so no configuration is required and sites without the extension behave as before.

IgbinarySerialize distinguishes its payloads from plain serialize() output using an internal binary prefix, so existing rows written by older Drupal versions remain decodable after upgrade. There is no data migration; legacy rows are simply re-encoded with igbinary the next time they are written.

The serialization.phpserialize service is unchanged and still available for code that explicitly needs PHP's native serialize().

**Note:** This change concerns the *object-aware* serializer used by cache and key/value storage. It is unrelated to the serializer service from the serialization module (Symfony Serializer used by REST/JSON:API), and unrelated to serialization.json / serialization.yaml, which handle text formats.

**What contrib and custom code should do:**

**If you inject the serializer, prefer the interface, not a concrete service ID.** Doing so makes your code respect whatever core or the site has chosen as the default, and lets sites swap implementations without patching you.

**Recommended:**

Inject by the interface — autowiring picks up the current default automatically:

```php
use Drupal\Component\Serialization\ObjectAwareSerializationInterface;

public function __construct(
  private readonly ObjectAwareSerializationInterface $serializer,
) {}
```

Or, if you write services.yml by hand, reference the interface as the service ID:

```php
my_module.thing:
  class: Drupal\my_module\Thing
  arguments: ['@Drupal\Component\Serialization\ObjectAwareSerializationInterface']
```

**Discouraged:**

Do **not** hard-code @serialization.phpserialize if your intent is "the default Drupal serializer." That ID still resolves to plain PHP serialize() and will not benefit from igbinary even on hosts where it is available. Only reference it explicitly when you specifically need PHP's serialize() format (for example, when interoperating with stored payloads outside Drupal that you know are plain PHP-serialized).

Likewise, do not hard-code @serialization.igbinary unless you specifically require the hybrid igbinary-or-fallback behavior. Pinning to a concrete service prevents sites and contrib from substituting their own implementation through the interface alias.

**Why no serializer.default alias?:**

The interface (Drupal\Component\Serialization\ObjectAwareSerializationInterface) already serves as the canonical way to refer to "the default object-aware serializer." Adding a second alias such as serializer.default would only duplicate what the interface alias already provides and split contrib between two equivalent spellings. The interface alias is the documented contract; everything that wants the swappable default should depend on it.

**Replacing the serializer entirely:**

To replace the default with your own implementation site-wide, override the alias in a ServiceProvider or services.yml, e.g.:

```php
services:
  Drupal\Component\Serialization\ObjectAwareSerializationInterface: '@my_module.my_serializer'
```

Any consumer that injects the interface — including core's database cache and expirable key/value backends — will pick up the replacement.

**Affected core consumers:**

The following core services were updated to inject the interface rather than @serialization.phpserialize:

- cache.backend.database (Drupal\Core\Cache\DatabaseBackendFactory)

- keyvalue.expirable.database (Drupal\Core\KeyValueStore\KeyValueDatabaseExpirableFactory)

- The early-bootstrap cache container in Drupal\Core\DrupalKernel (uses serialization.igbinary directly because the bootstrap container is built before the full alias graph)

**BC and upgrade notes:**

- No update path is required. Existing serialized rows in cache_* and key_value_expire tables remain readable.

- Sites without the igbinary extension are not affected at runtime; encoding falls back to serialize().

- No public API was removed or deprecated.

---

## 194. [theme-settings.php for hook_form_system_theme_settings_alter() has been deprecated](https://www.drupal.org/node/3587273)

- **Node ID**: [3587273](https://www.drupal.org/node/3587273)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3580152](https://www.drupal.org/node/3580152)

**Description:**

theme-settings.php files have been deprecated.

11.3+
Convert the hook to OOP.
Before 11.3:
Move the hook to the .theme file.

---

## 195. [NavigationShortcutsBlock is deprecated](https://www.drupal.org/node/3587759)

- **Node ID**: [3587759](https://www.drupal.org/node/3587759)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3581816](https://www.drupal.org/node/3581816)

**Description:**

The 'navigation_shortcuts' block has moved from the Navigation module to the Shortcut module.

Therefor \Drupal\navigation\Plugin\Block\NavigationShortcutsBlock is deprecated.  Instead, use \Drupal\shortcut\Plugin\Block\ShortcutNavigationBlock.

---

## 196. [Functions in update.compare.inc are deprecated](https://www.drupal.org/node/3587768)

- **Node ID**: [3587768](https://www.drupal.org/node/3587768)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3580705](https://www.drupal.org/node/3580705)

**Description:**

The functions in update.compare.inc have been deprecated and moved to Drupal\update\UpdateCalculate

**Before:****update_process_project_info(&$projects)
After:****\Drupal::service(UpdateCalculate::class)-&gt;processProjectInfo($projects)
Before:****update_calculate_project_data($available)
After:****\Drupal::service(UpdateCalculate::class)-&gt;projectData($available)
Before:****update_calculate_project_update_status(&$project_data, $available)
After:**
\Drupal::service(UpdateCalculate::class)-&gt;projectUpdateStatus($project_data, $available)

---

## 197. [EntityTypeInterface::getOriginalClass method is deprecated](https://www.drupal.org/node/3587853)

- **Node ID**: [3587853](https://www.drupal.org/node/3587853)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3557461](https://www.drupal.org/node/3557461)

**Description:**

Since [#3127026: Not possible to override an entity type class multiple times](/project/drupal/issues/3127026), there are no more usages of the EntityTypeInterface::getOriginalClass method so it is now deprecated.

A new getDecoratedClasses method had been added in case your code needs to check for an entity type class.
**Before:**

```php
$entity_type->getOriginalClass() == $class_name
```

**After:**

```php
in_array($class_name, $entity_type->getDecoratedClasses(), TRUE)
```

The new method supports any number of overrides.

---

## 198. [Entity reference selection handlers now check create access before auto-creating entities](https://www.drupal.org/node/3587988)

- **Node ID**: [3587988](https://www.drupal.org/node/3587988)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3372919](https://www.drupal.org/node/3372919)

**Description:**

The Drupal\Core\Entity\Plugin\EntityReferenceSelection\DefaultSelection plugin (the default selection handler used by entity reference fields) now verifies that the current user has create access for the target bundle before auto-creating a referenced entity from an autocomplete widget.

As most selection handlers inherit from this one, this technically would apply to all of those.

Previously, when a field was configured with "Create referenced entities if they don't already exist", any user able to edit the field could trigger creation of the target entity, regardless of whether they had permission to create that entity type/bundle directly.

The "Create referenced entities if they don't already exist" checkbox in the entity reference field UI gains a companion checkbox: "Respect create access, only userswith the necessary permission can create new entities."

A new boolean configuration key, auto_create_check_access, has been added to the selection handler configuration:

- For new fields, it defaults to TRUE (access is checked).

- For existing fields on sites upgraded from earlier versions, it is kept as FALSE to preserve existing behavior. Site builders are encouraged to flip it to TRUE after reviewing each field.

For further hardening, when the flag is TRUE and the current user lacks create access for the target bundle, DefaultSelection::createNewEntity() now throws Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException. If you were calling this code directly, you might want to verify this is handled as you expect.

---

## 199. [The check_markup() function is deprecated](https://www.drupal.org/node/3588040)

- **Node ID**: [3588040](https://www.drupal.org/node/3588040)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#455724](https://www.drupal.org/node/455724)

**Description:**

The check_markup() function is deprecated and no replacement is provided. It's always recommended to return a renderable array, when possible, without flattening as markup, to preserve the cacheability metadata.

**Before:**

```php
$formatted_text = check_markup($text, 'basic_html', $langcode, $filter_types_to_skip);
```

**After:**

```php
$formatted_text = [
  '#type' => 'processed_text',
  '#text' => $text,
  '#format' => 'basic_html',
  '#filter_types_to_skip' => $filter_types_to_skip,
  '#langcode' => $langcode,
];
```

Particular cases:

- Mail: A common use case where check_markup() is still frequently used is hook_mail() as that does not support render arrays. Future Drupal mail system development will pipe the mail preparation through Twig (for both, plain and HTML messages). This will remove the need to manually render the renderable array (see [#3539651: Introduce email plugins](/project/drupal/issues/3539651)). Until that API is available,  \Drupal::service('renderer')-&gt;renderInIsolation($build) may be used. Refer to the documentation on \Drupal\Core\Render\RendererInterface when render() vs renderInIsolation() should be used.

- Tests: Many tests are only asserting that a text has been formatted correctly. For convenience, Drupal provides the \Drupal\Tests\filter\Traits\ProcessedTextTestTrait trait with a helper method, facilitating testing.

---

## 200. [Return types have changed on some JSON:API Normalizer methods](https://www.drupal.org/node/3588047)

- **Node ID**: [3588047](https://www.drupal.org/node/3588047)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3563533](https://www.drupal.org/node/3563533)

**Description:**

The following methods have had changes to their return type declarations:

***Drupal\serialization\Normalizer\ComplexDataNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): array
***Drupal\serialization\Normalizer\ConfigEntityNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): array
***Drupal\serialization\Normalizer\ContentEntityNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): array
***Drupal\serialization\Normalizer\EntityReferenceFieldItemNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): array
***Drupal\serialization\Normalizer\ListNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): array
***Drupal\serialization\Normalizer\MarkupNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): string
***Drupal\serialization\Normalizer\NullNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): null
***Drupal\serialization\Normalizer\TimestampItemNormalizer***

**Before**: public function normalize($object, $format = NULL, array $context = []): array|string|int|float|bool|\ArrayObject|NULL**After**: public function normalize($object, $format = NULL, array $context = []): array

---

## 201. [SDC library overrides now support a fonts key for preloading](https://www.drupal.org/node/3588509)

- **Node ID**: [3588509](https://www.drupal.org/node/3588509)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3586470](https://www.drupal.org/node/3586470)

**Description:**

Similar to [https://www.drupal.org/node/3585574](https://www.drupal.org/node/3585574).

Now this support is also added to libraryOverrides inside component definition:

```php
libraryOverrides:
  ...
  fonts:
    component-font.woff2:
      preload: true
```

---

## 202. [ExtensionList ::getList() now has an optional $skip_cache argument](https://www.drupal.org/node/3588598)

- **Node ID**: [3588598](https://www.drupal.org/node/3588598)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3588490](https://www.drupal.org/node/3588490)

**Description:**

ExtensionList::getList() now has a new, optional, $skip_cache = FALSE argument. When this argument is passed, both persistent and static caching will be bypassed and the list of extensions will be recalculated from the file-system. Note that as well as bypassing the cache, this also does not update the cache either, although it will populate the static cache so that subsequent calls on the same page will use the freshed possible information.

This can be used in situations that previously would have invalidated the cache in order to get a full list of available extensions:

Before:

```php
// Get all available themes.
    $themes = $this->themeExtensionList->reset()->getList();
```

After:

```php
// Get all available themes.
    $themes = $this->themeExtensionList->getList(TRUE);
```

---

## 203. [Route preloading/caching changes](https://www.drupal.org/node/3589089)

- **Node ID**: [3589089](https://www.drupal.org/node/3589089)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3503843](https://www.drupal.org/node/3503843)

**Description:**

The internal route-by-name-lookup cache is changed to use a new dedicated fast-chained cache bin "routes", that caches all requested routes in APCu if available. It is recommended that especially larger sites monitor their APCu usage and expunge and consider increasing the available apcu memory to account for this.

Before 11.4, Drupal core contained a RoutePreloader event subscriber, that preloaded all routes that were identified as non-admin HTML and preloaded them on every HTML request format request.

Sites can have hundreds of these routes, and this preloading often resulted in significant additional memory usage. Due to render caching, most page requests only require a small subset of routes. That event subscriber has been removed.

For now, the \Drupal\Core\Routing\PreloadableRouteProviderInterface remains while possible alternative improvements to preloading are explored but it may be deprecated in the future.

---

## 204. [object JSON schema type is no longer added to every SDC prop](https://www.drupal.org/node/3589343)

- **Node ID**: [3589343](https://www.drupal.org/node/3589343)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3554720](https://www.drupal.org/node/3554720)

**Description:**

SDC API is not adding the object JSON schema type to all props anymore. So, such component prop:

```php
heading_level:
  title: "Heading level"
  type: integer
```

is not automatically transformed into:

```php
heading_level:
  title: "Heading level"
  type: ['integer', 'object']
```

This has no impact for themers but module developers will not need to circumvent this addition anymore. Example:

Before
After

```php
$form['value'] = [
  '#type' => 'number',
   '#step' => 0.01,
];
if (is_array($schema['type'])
  && in_array('integer', $schema['type'])) {
  $form['value']['#step'] = 1;
}
```

```php
$form['value'] = [
  '#type' => 'number',
   '#step' => 0.01,
];
if ($schema['type'] === 'integer')  {
  $form['value']['#step'] = 1;
}
```

If a mechanism relies on this addition, it must be adapted.

---

## 205. [\Drupal\node\Controller\NodeViewController is deprecated](https://www.drupal.org/node/3589636)

- **Node ID**: [3589636](https://www.drupal.org/node/3589636)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3589630](https://www.drupal.org/node/3589630)

**Description:**

\Drupal\node\Controller\NodeViewController is deprecated, it was functionally equivalent to EntityViewController.

Use \Drupal\Core\Entity\Controller\EntityViewController instead.

---

## 206. [All batch related functions in locale.batch.inc, locale.bulk.inc and locale.compare.inc have been deprecated](https://www.drupal.org/node/3589759)

- **Node ID**: [3589759](https://www.drupal.org/node/3589759)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3581303](https://www.drupal.org/node/3581303)

**Description:**

Deprecated procedural callback
Replacement

locale_translation_batch_version_check
\Drupal::service(LocaleFetch::class)-&gt;batchVersionCheck()

locale_translation_batch_status_check
\Drupal::service(LocaleFetch::class)-&gt;batchStatusCheck()

locale_translation_batch_fetch_download
\Drupal::service(LocaleFetch::class)-&gt;batchDownload()

locale_translation_batch_fetch_import
\Drupal::service(LocaleFetch::class)-&gt;batchImport()

locale_translation_batch_fetch_finished
\Drupal::service(LocaleFetch::class)-&gt;batchFinished()

_locale_translation_batch_status_operations
\Drupal::service(LocaleFetch::class)-&gt;getStatusOperations()

locale_translate_batch_build
\Drupal::service(LocaleImportBatch::class)-&gt;buildBatch()

locale_translate_batch_import
\Drupal::service(LocaleImportBatch::class)-&gt;batchImport()

locale_translate_batch_import_save
\Drupal::service(LocaleImportBatch::class)-&gt;batchSave()

locale_translate_batch_refresh
\Drupal::service(LocaleImportBatch::class)-&gt;batchRefresh()

locale_translate_batch_finished
\Drupal::service(LocaleImportBatch::class)-&gt;batchFinished()

locale_config_batch_update_components
\Drupal::service(LocaleConfigBatch::class)-&gt;buildBatch()

locale_config_batch_build
No replacement it was inlined into LocaleConfigBatch::buildBatch

locale_config_batch_update_default_config_langcodes
\Drupal::service(LocaleConfigBatch::class)-&gt;batchUpdateDefaultConfigLangcodes()

locale_config_batch_update_config_translations
\Drupal::service(LocaleConfigBatch::class)-&gt;batchUpdateConfigTranslations()

locale_config_batch_finished
\Drupal::service(LocaleConfigBatch::class)-&gt;batchFinished()

locale_translation_check_projects_batch
\Drupal::service(LocaleProjectChecker::class)-&gt;triggerBatch()

locale_translation_batch_status_build
No replacement it was inlined into LocaleProjectChecker::batchCheckProjects

locale_translation_batch_status_finished
\Drupal::service(LocaleProjectChecker::class)-&gt;batchFinished()

---

## 207. [Node search plugin node_search moved to sub-module Search Node in Search](https://www.drupal.org/node/3590298)

- **Node ID**: [3590298](https://www.drupal.org/node/3590298)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3587564](https://www.drupal.org/node/3587564)

**Description:**

The plugin node_search is now in the new sub-module of the Search module, search_node.

Any class extending \Drupal\node\Plugin\Search\NodeSearch should change to extending \Drupal\search_node\Plugin\Search\SearchNode

---

## 208. [Site directory and app root are now in AppContext provided to DrupalKernel from DrupalRuntime](https://www.drupal.org/node/3590362)

- **Node ID**: [3590362](https://www.drupal.org/node/3590362)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3590337](https://www.drupal.org/node/3590337)

**Description:**

---

## 209. [locale_get_plural() is deprecated, replaced by PluralIndexInterface service](https://www.drupal.org/node/3590542)

- **Node ID**: [3590542](https://www.drupal.org/node/3590542)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2660338](https://www.drupal.org/node/2660338)

**Description:**

The function locale_get_plural() is deprecated.

**Before**:

```php
locale_get_plural($count, $langcode);
```

After:

```php
\Drupal::service(PluralIndexInterface::class)->getPluralIndex($count, $langcode)
```

This is usually not called directly and is internally used to display plural translatable strings through \Drupal\Core\StringTranslation\PluralTranslatableMarkup.

---

## 210. [The Article and Page content types are removed from the Standard profile and recipe](https://www.drupal.org/node/3590571)

- **Node ID**: [3590571](https://www.drupal.org/node/3590571)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3587118](https://www.drupal.org/node/3587118)

**Description:**

The Article and Page content types are not installed on new sites when using the Standard install profile or recipe.

Sites must configure the content types they require.

---

## 211. [Locale translation status functions deprecated in favor of the LocaleSource service](https://www.drupal.org/node/3591660)

- **Node ID**: [3591660](https://www.drupal.org/node/3591660)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3590050](https://www.drupal.org/node/3590050)

**Description:**

The following functions have been deprecated:

- locale_translation_get_status

- locale_translation_status_save

- locale_translation_status_delete_languages

- locale_translation_status_delete_projects

- locale_translation_clear_status

Translation status was a different concept from sources, which is something that has already been partially refactored into the [LocaleSource service](/node/3569330). The objects were however the same thing, and calls were made into either direction. locale_translation_get_status() called buildSource() and loadSources() called locale_translation_get_status(). All remaining translation status related functions have been deprecated now and merged into the existing LocaleSource service. loadSources() and loadSource() now only load the specific projects that have been requested. They also always return a source for the requested projects/languages, if there is no stored source, one is built automatically.

A source is the information about the import/translation/po file status of a project for a specific language.

Before:

```php
locale_translation_get_status()
```

After:

```php
// All or multiple projects/languages.
\Drupal::service(LocaleSource::class)->loadSources()
// Recommended: Load a specific source for a given project and language combination.
\Drupal::service(LocaleSource::class)->loadSource($project, $langcode)
```

Before:

```php
locale_translation_status_save()
```

After:

```php
\Drupal::service(LocaleSource::class)->saveSource()
```

Before:

```php
locale_translation_status_delete_languages()
```

After:

```php
\Drupal::service(LocaleSource::class)->deleteSourcesByLanguage()
```

Before:

```php
locale_translation_status_delete_projects()
```

After:

```php
\Drupal::service(LocaleSource::class)->deleteSources($projects)
```

Before:

```php
locale_translation_clear_status()
```

After:

```php
\Drupal::service(LocaleSource::class)->clearSources()
```

There is a new LocaleTranslationSource value object for managing source operations, it has the same public properties as the previous untyped stdClass objects.
There is a new \Drupal::service(LocaleSource::class)-&gt;updateLastChecked() and \Drupal::service(LocaleSource::class)-&gt;getLastChecked for managing the global last checked time, the previous state key is no longer maintained.

---

## 212. [update_fetch_with_http_fallback setting is deprecated](https://www.drupal.org/node/3591920)

- **Node ID**: [3591920](https://www.drupal.org/node/3591920)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3591513](https://www.drupal.org/node/3591513)

**Description:**

The update_fetch_with_http_fallback setting is deprecated in drupal:11.4.0 and will be removed in drupal:12.0.0.

This setting allowed sites to fall back to HTTP when HTTPS requests to updates.drupal.org failed. Using HTTP for update requests exposes sites to potential man-in-the-middle attacks and should not be used.

Sites experiencing HTTPS connection issues should resolve them directly (e.g. by ensuring OpenSSL is properly configured) rather than relying on this insecure fallback.

**Updates required:**

Remove the update_fetch_with_http_fallback setting from your settings.php file:

**Before::**

$settings['update_fetch_with_http_fallback'] = TRUE;

**After::**

(line removed)

If your site relies on this setting because HTTPS connections to updates.drupal.org are failing, resolve the underlying SSL/TLS issue instead.
See [https://www.drupal.org/docs/system-requirements/php-requirements#openssl](https://www.drupal.org/docs/system-requirements/php-requirements#openssl)

---

---

## 213. [hook_preprocess_views_view_grouping() is now invoked for all grouping levels, including single-level grouping](https://www.drupal.org/node/3592452)

- **Node ID**: [3592452](https://www.drupal.org/node/3592452)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#2950737](https://www.drupal.org/node/2950737)

**Description:**

hook_preprocess_views_view_grouping() was only invoked for nested (multi-level) grouping. For single-level grouping, StylePluginBase::renderGroupingSets() bypassed the views_view_grouping theme hook and delegated directly to renderRowGroup(), which uses the style's own theme (e.g. views_view_unformatted). The preprocess hook was therefore never triggered.

Two changes have been made to fix this:

- **StylePluginBase::renderGroupingSets():**

At the leaf level (rows are individual result rows, not nested groups), when grouping is configured, the method now builds a #theme =&gt; views_view_grouping render array instead of calling renderRowGroup() directly. This ensures hook_preprocess_views_view_grouping() is invoked at every grouping level.

Before:

```php
// Leaf rows always went through renderRowGroup(), skipping views_view_grouping.
$single_output = $this->renderRowGroup($set['rows']);
```

After:

```php
if (!empty($this->options['grouping'])) {
  $single_output = [
    '#theme' => $theme_functions,
    '#view' => $this->view,
    '#grouping' => $this->options['grouping'][$level] ?? [],
    '#rows' => $set['rows'],
  ];
}
else {
  $single_output = $this->renderRowGroup($set['rows']);
}
```

- **ViewsThemeHooks::preprocessViewsViewGrouping():**

The preprocess function now detects whether $variables['rows'] contains nested grouping sets (arrays with a 'group' key) or already-rendered rows (leaf level), and handles each case:

**Non-leaf level:** calls renderGroupingSets() recursively, as before.

- **Leaf level (including single-level grouping):** wraps the rows in the style's own theme render array.

$variables['content'] is now always an **indexed array of render arrays** at every level.

Additionally, $variables['title'] is now wrapped in a Markup object to prevent Twig from double-escaping the already-HTML-escaped rendered field value used as the group title.

**Impact on site builders and module developers:**

This change affects you if you have:

- A custom views-view-grouping.html.twig template — it will now also be invoked for **single-level grouping**, where it was previously skipped. Review your template to ensure it handles both leaf and non-leaf levels correctly.

- A hook_preprocess_views_view_grouping() implementation — it will now fire for all grouping levels. The $variables['content'] variable is always an indexed array of render arrays; if your hook assumed a different structure for single-level grouping, update it accordingly.

- A hook_preprocess_views_view_grouping() implementation that outputs $variables['title'] — it is now a Markup object; do not re-escape it.

---

## 214. [JavaScript translation files are now stored in the assets:// path location](https://www.drupal.org/node/3592731)

- **Node ID**: [3592731](https://www.drupal.org/node/3592731)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3591076](https://www.drupal.org/node/3591076)

**Description:**

Drupal 10.1.0 [introduced the assets:// stream wrapper](https://www.drupal.org/node/3328126) to store aggregates.

It now also stores the aggregation files generated by the locale module for translations used in JavaScript files.

---

## 215. [New cache.file_parsing bin and file parsing cache collector](https://www.drupal.org/node/3593451)

- **Node ID**: [3593451](https://www.drupal.org/node/3593451)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3486503](https://www.drupal.org/node/3486503)

**Description:**

A new cache.file_parsing cache bin has been added, for persistent caching of the result of file parsing. This bin does not have the cache.bin service tag so that it persists across cache clears (whether via drush, the UI or programmatic calls to drupal_flush_all_caches(). However the bin can be manually cleared if necessary using the CacheBackendInterface::deleteAll().

The bin is designed to be used with implementations of the new Drupal\Core\Utility\FileParsingCacheCollectorBase which supports validation of individual file entries via mtime checks.

This can be used as a replacement of the existing FileCache system which implements similar logic but with APCu as a storage backend. By using a persistent cache instead of APCu, the cache state will always be consistent across cli and between multiple web servers, which should improve response times both for local development which often involves frequent cache clears via the cli, and on production sites immediately after a full cache clear such as via deployments.

There is also a YamlCacheCollector class, which is used for asset library parsing (libraries.yml) and route parsing (routing.yml).

More implementations may be added to core in further issues.

---

## 216. [Various properties on LocaleTranslatableProject, LocaleTranslationSource, LocaleFile deprecated](https://www.drupal.org/node/3593802)

- **Node ID**: [3593802](https://www.drupal.org/node/3593802)
- **Target Version**: `11.4.0` (Branch: `11.x`)
- **Related Issues**: [#3593731](https://www.drupal.org/node/3593731)

**Description:**

---

## 217. [The $entity_type parameter of EntityRouteProviderInterface::getRoutes() is deprecated](https://www.drupal.org/node/3593831)

- **Node ID**: [3593831](https://www.drupal.org/node/3593831)
- **Target Version**: `11.4.0` (Branch: `11.5.x`)
- **Related Issues**: [#2977140](https://www.drupal.org/node/2977140)

**Description:**

**Overview:**

As of Drupal 11.4.0, the $entity_type parameter on EntityRouteProviderInterface::getRoutes() is deprecated. Route providers already receive the entity type on construction, so passing it again is redundant. Core no longer passes the argument; providers should read the entity type from the value injected on construction. The parameter is removed in drupal:13.0.0.

**Changes:**

```php
// Before.
public function getRoutes(EntityTypeInterface $entity_type) {
  // ...
}
$routes = $route_provider->getRoutes($entity_type);

// After.
public function getRoutes(?EntityTypeInterface $entity_type = NULL) {
  $entity_type = $entity_type ?? $this->entityType;
  // ...
}
$routes = $route_provider->getRoutes();
```

---

## 218. [SYMFONY_DEPRECATIONS_HELPER environment variable is deprecated](https://www.drupal.org/node/3594014)

- **Node ID**: [3594014](https://www.drupal.org/node/3594014)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3589108](https://www.drupal.org/node/3589108)

**Description:**

The SYMFONY_DEPRECATIONS_HELPER environment variable is no longer used to setup test runs' deprecation reporting, and its use is deprecated.

Its logic has been replaced by the DeprecationHandler extension, that needs to be configured in phpunit.xml as follows:

```php











    ....


```

---

## 219. [Calling AuthenticationSubscriber constructor without authenticationCollector argument is deprecated](https://www.drupal.org/node/3594314)

- **Node ID**: [3594314](https://www.drupal.org/node/3594314)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3546804](https://www.drupal.org/node/3546804)

**Description:**

The constructor AuthenticationSubscriber has a new parameter $authenticationCollector, of type Drupal\Core\Authentication\AuthenticationCollectorInterface. This is to allow error messages to include the specific authentication method.

This parameter is optional from Drupal 11.4.0, but will be mandatory in Drupal 12.0.0.

---

## 220. [ViewExecutable::getHandler() is deprecated, use ViewExecutable::getHandlerConfiguration() instead](https://www.drupal.org/node/3598241)

- **Node ID**: [3598241](https://www.drupal.org/node/3598241)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3034692](https://www.drupal.org/node/3034692)

**Description:**

\Drupal\views\ViewExecutable::getHandler() has been renamed to**\Drupal\views\ViewExecutable::getHandlerConfiguration(). The old method is
deprecated and will be removed in Drupal 13.
The old name was misleading: despite its name, the method does not return a handler
plugin instance — it returns the stored *configuration array* of a handler on a
given display. This was easily confused with
\Drupal\views\Plugin\views\display\DisplayPluginBase::getHandler(), which
does return an actual handler plugin object. The new name makes the distinction clear.
The behavior and signature of the method are unchanged; only the name is different.
Before:**

```php
$configuration = $view->getHandler('page_1', 'field', 'title');
```

**After:**

```php
$configuration = $view->getHandlerConfiguration('page_1', 'field', 'title');
```

Calling ViewExecutable::getHandler() still works but now triggers a**deprecation error:

```php
Drupal\views\ViewExecutable::getHandler() is deprecated in drupal:11.4.0 and is removed
from drupal:13.0.0. Use self::getHandlerConfiguration() instead.
```

Note that the related methods are not** affected by this change:

- ViewExecutable::getHandlers() — still returns all handler configurations of a type on a display.

- ViewExecutable::setHandler() / setHandlerOption() / removeHandler() — unchanged.

- DisplayPluginBase::getHandler() — unchanged; this one returns a handler plugin instance.

---

## 221. [drupal/core-recommended allows minor updates of dependencies](https://www.drupal.org/node/3601304)

- **Node ID**: [3601304](https://www.drupal.org/node/3601304)
- **Target Version**: `11.4.0` (Branch: `11.4.x`)
- **Related Issues**: [#3600889](https://www.drupal.org/node/3600889)

**Description:**

drupal/core-recommended no longer pins versions of the following dependencies of Drupal core. This means that sites will be able to apply security updates for these dependencies immediately upon release (as long as they do not require a new major version).

Since these dependencies may *not* have been tested with Drupal core yet, site owners should ensure adequate quality assurance (QA) occurs before these are deployed to production.

- guzzlehttp/guzzle

- guzzlehttp/promises

- guzzlehttp/psr7

- symfony/polyfill-ctype

- symfony/polyfill-iconv

- symfony/polyfill-intl-grapheme

- symfony/polyfill-intl-idn

- symfony/polyfill-intl-normalizer

- symfony/polyfill-mbstring

- symfony/polyfill-php86

- twig/html-extra

- twig/twig

Previously, drupal/core-recommended pinned versions of these dependencies to minor versions, to ensure that a new minor version of a dependency, which can include unintended or 'internal' API changes, would not be installed until it had gone through Drupal core's testing and release process.

Since [Composer 2.9](https://blog.packagist.com/composer-2-9/), if a dependency issues a security release which is only available in a new minor version, composer actively blocks any installs or updates until the user either changes the version constraint, aliases, or allow-lists the affected version.

Simultaneously, several of Drupal core's dependencies have had security releases only available in new minor versions. The combination of these security release policies and composer's new behavior has meant that site using drupal/core-recommended could not update immediately. They could only update after a new release of Drupal core that included updating the constraints for drupal/core-recommended to new minor for the affected dependencies.

To mitigate this, drupal/core-recommended no longer constrains dependencies for the dependencies
that recently made security releases on a new minor release.  Instead drupal/core-recommended will  have the same version constraints as drupal/core for the dependencies listed above.
Remember, site owners should ensure adequate quality assurance (QA) occurs before these are deployed to production.

---
