# Drupal Change Records for 11.3.0

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

## [Widget elements can be written using object oriented approach](https://www.drupal.org/node/3532733)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

[A new object-oriented API has been added for working with form and render](https://www.drupal.org/node/3532720) and widgets can now written with fully object oriented code without writing any arrays, without `#`.


Before:



```php
public function formElement(FieldItemListInterface $items, $delta, array $element, array &$form, FormStateInterface $form_state) {
    $element['value'] = $element + [
      '#type' => 'textfield',
      '#default_value' => $items[$delta]->value ?? NULL,
      '#size' => $this->getSetting('size'),
      '#placeholder' => $this->getSetting('placeholder'),
      '#maxlength' => $this->getFieldSetting('max_length'),
      '#attributes' => ['class' => ['js-text-full', 'text-full']],
    ];

    return $element;
  }
```

After:



```php
public function singleElementObject(FieldItemListInterface $items, $delta, Widget $widget, ElementInterface $form, FormStateInterface $form_state): ElementInterface {
    $value = $widget->createChild('value', Textfield::class, copyProperties: TRUE);
    $value->default_value = $items[$delta]->value ?? NULL;
    $value->size = $this->getSetting('size');
    $value->placeholder = $this->getSetting('placeholder');
    $value->maxlength = $this->getFieldSetting('max_length');
    $value->attributes = ['class' => ['js-text-full', 'text-full']];
    return $widget;
  }
```

Note this does not call `parent::singleElementObject()` and that's correct. `WidgetBase::singleElementObject` provides backwards compatibility by calling `$this->formElement()` and wrapping it for the modern OOP elements API, there's no need to call it once the conversion above has been done.

---

## [New Object oriented approach for working with form/render arrays](https://www.drupal.org/node/3532720)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

A new object-oriented API has been added for working with form and render elements.


The API builds on top of existing Render/Form element plugins and supports casting to and from an object.


The objects are provided as a convenience for working with render arrays and provide IDE integration so that supported keys can be auto-completed.


Once the form/render arrays have been modified using the new API, in the first instance they must be cast back to render arrays to continue to work with Drupal. Future releases will build support for these objects as first class returns.


Examples

In all the examples below `$elementInfoManager` is the element info plugin manager.
This can be retrieved from the service container using `\Drupal::service(Drupal\Core\Render\ElementInfoManagerInterface::class)` or alternatively injected using available dependency injection approaches.
If you're working inside a class that extends from `\Drupal\Core\Form\FormBase`, this is also available as `$this->elementInfoManager()`


Create a new submit button

```php
$submit = $elementInfoManager->fromClass(\Drupal\Core\Render\Element\Submit::class))
$submit->value = $this->t('Submit');
```

If you're using an IDE, you should get auto-complete suggestions on the properties you can use on the object





Alter an existing form
Adding a new child element.



```php
$form_obj = $elementInfoManager->fromRenderable($form);
$checkbox  = $form_obj->createChild('field_name', \Drupal\Core\Render\Element\Checkbox::class);
$checkbox->title = $this->t('Do you like this new API?');
$checkbox->required = TRUE;
```

Removing a child element



```php
$form_obj = $elementInfoManager->fromRenderable($form);
// No soup for you.
$form_obj->removeChild('soup');
```
Iterating over children

```php
$form_obj = $elementInfoManager->fromRenderable($form);
$titles = [];
foreach ($form_obj->getChildren() as $child) {
  // Children are objects too.
  $titles[] = $child->title;
}
```
Getting a single child

```php
$form_obj = $elementInfoManager->fromRenderable($form);
$form_obj->getChild('submit')->value = $this->t('Save');
```
Casting back to a render array
At this point in time, form build methods and the like still expect a render array.
So the new objects are only for developer convenience while building and working with form elements.
To cast the object back to a render array to be returned from a form builder function, use the `toRenderable` method



```php
public function buildForm(array $form, FormStateInterface $form_state): array {
  $form_obj = $this->elementInfoManager()->fromRenderable($form);
  $submit =  $form_obj->createChild('submit');
  $submit->value = $this->t('Submit');
  return $form_obj->toRenderable();
}
```
Adding support for custom element plugins
Adding support to a custom/contrib element plugin is a matter of ensuring that any custom # keys are documented as `@property` keys on the plugin class.


See [an example](https://git.drupalcode.org/project/drupal/-/blob/17ea568b9a3fa8602f7cf2f69edfcee54edd2dab/core/lib/Drupal/Core/Entity/Element/EntityAutocomplete.php#L23) from core

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

---

## [A new database driver (mysqli) for MySQL/MariaDB for parallel queries](https://www.drupal.org/node/3516913)

- **Version**: 11.3.0
- **Branch**: 11.x
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
- **Branch**: 11.x
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

## [Transaction::commitOrRelease() method introduced to explicity commit a transaction](https://www.drupal.org/node/3512006)

- **Version**: 11.3.0
- **Branch**: 11.x
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

## [Experimental Symfony Mailer Module](https://www.drupal.org/node/3519253)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

An experimental mailer module has been added which puts the necessary services in place such that bleeding-edge contrib and custom code can start using the mail delivery part of [Symfony Mailer](https://symfony.com/doc/current/mailer.html) by simply retrieving / referencing a configured mailer service from / in the container.


The new mail API is subject to breaking changes. This change record will be updated with every iteration.


Setup
Enable the experimental `mailer` module. Then configure the transport [DSN](https://symfony.com/doc/current/mailer.html#using-built-in-transports) by specifying its components in the `mailer_dsn` config. The following examples may serve as a starting point:



- For the default sendmail transport:

```php
$config['system.mail']['mailer_dsn'] = [
  'scheme' => 'sendmail',
  'host' => 'default',
];
```


- For mailpit on localhost:

```php
$config['system.mail']['mailer_dsn'] = [
  'scheme' => 'smtp',
  'host' => 'localhost',
  'port' => 1025,
];
```


- For authenticated SMTP:

```php
$config['system.mail']['mailer_dsn'] = [
  'scheme' => 'smtp',
  'host' => 'smtp.example.com',
  'user' => 'some-username@example.com',
  'password' => 'correct horse battery staple',
  'options' => [
    'local_domain' => 'example.org',
  ],
];
```



Preliminary Mail Delivery API
See issue [#3379794: Add symfony mailer transports to Dependency Injection Container (mail delivery layer)](https://www.drupal.org/project/drupal/issues/3379794).



- Service: `Symfony\Component\Mailer\MailerInterface`:
Custom and contrib modules may use this service to pass mails to the mail delivery layer. This is the main entry point for the mail delivery layer.


- Service: `Symfony\Component\Mailer\Transport\TransportInterface`:
Custom and contrib modules may use this service to directly inject mails to the configured mail transport for delivery. This will skip Symfony messenger (if configured). This should only be necessary in advanced use cases.

- Service: `Drupal\Core\Mailer\TransportServiceFactoryInterface`:
Custom and contrib modules may decorate or replace this service in order to customize construction of `Symfony\Component\Mailer\Transport\TransportInterface`. This should only be necessary in advanced use cases. E.g., if certain messages are sent via dedicated transports.

- Events [MessageEvent](https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/Mailer/Event/MessageEvent.php), [SentMessageEvent](https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/Mailer/Event/SentMessageEvent.php) and [FailedMessageEvent](https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/Mailer/Event/FailedMessageEvent.php):
Custom and contrib modules may register event subscribers to act on emails before and after they are sent.

- Abstract service `Symfony\Component\Mailer\Transport\AbstractTransportFactory` and service tag `mailer.transport_factory`:
Custom and contrib modules may supply third-party or custom transport factories using `Symfony\Component\Mailer\Transport\AbstractTransportFactory` as their parent service, tagged with `mailer.transport_factory` and typically with `Symfony\Component\Mailer\Transport\AbstractTransportFactory` as their parent class.

- The `mailer_sendmail_commands` setting:
An array of command lines which are allowed as the `command` option in the sendmail transport.


Example: Send a Message
The following example can serve as a starting point for experiments with the Symfony Mailer mail delivery API:



```php
// Get the mailer instance from the container.
$mailer = $container->get(\Symfony\Component\Mailer\MailerInterface::class);

// Create a new message.
$email = new \Symfony\Component\Mime\Email();
$email->subject("Test message")
  ->from('test@localhost.localdomain')
  ->text('Hello test runner!');

try {
  $mailer->send($email->to('admin@localhost.localdomain'));
  // $messenger->addStatus($this->t('Sent test message'));
}
catch (RuntimeException $e) {
  // $messenger->addError($this->t('Failed to send test message'));
}
```
Example: Register a Third-Party Transport
Contrib and custom modules providing third-party transports supply their own service tagged with the `mailer.transport_factory` tag, deriving from the abstract transport class:



```php
services:
  # Example of third-party service integration, requires symfony/google-mailer
  # https://packagist.org/packages/symfony/google-mailer
  Symfony\Component\Mailer\Bridge\Google\Transport\GmailTransportFactory:
    parent: Symfony\Component\Mailer\Transport\AbstractTransportFactory
    tags:
      - { name: mailer.transport_factory }
```
Preliminary Mail Building API
Not designed/implemented yet.

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

