# Drupal Change Records

## [run-tests.sh uses PHPUnit's API to determine the tests to run](https://www.drupal.org/node/3530388)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh</code> was using a bespoke algorithm to determine the list of tests to run; this was a legacy of simpletest times.</p>
<p>PHPUnit now has a public API that allows apps to wrap part of its routines, see <a href="https://docs.phpunit.de/en/10.5/extending-phpunit.html#wrapping-the-test-runner" rel="nofollow">Wrapping the Test Runner</a>.</p>
<p><code class=" language-php">run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh</code> has now been changed to discover the list of tests to execute using PHPUnit's API. This is a much more accurate process, because</p>
<ol>
<li>It deals natively with both legacy annotation and the new attributes for test metadata - thus effectively enabling the Drupal test codebase to use attributes</li>
<li>It executes the data providers methods, and then includes in the discovery list as many test cases as the number of datasets provided for a test method, thus determining the exact number of test cases to be run for each test class - which is relevant for ordering test execution sequence.</li>
<li>It shows details of warnings generated during the test discovery, which can be used to increase the test codebase quality - but it can cause discovered tests to be skipped from execution if they would fail.</li>
</ol>
<p>Some examples of warnings and how they can be addressed:</p>
<p><code class=" language-php"><span class="token operator">*</span> PHPUnit\<span class="token package">Event<span class="token punctuation">\</span>TestRunner<span class="token punctuation">\</span>WarningTriggered</span><span class="token punctuation">:</span> Cannot add file <span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>drupal<span class="token operator">/</span>core<span class="token operator">/</span>modules<span class="token operator">/</span>config<span class="token operator">/</span>tests<span class="token operator">/</span>config_test<span class="token operator">/</span>tests<span class="token operator">/</span>src<span class="token operator">/</span>Functional<span class="token operator">/</span>Rest<span class="token operator">/</span>ConfigTestJsonAnonTest<span class="token punctuation">.</span>php to test suite <span class="token string">"functional"</span> <span class="token keyword keyword-as">as</span> it was already added to test suite <span class="token string">"functional"</span></code></p>
<p>This is due to duplicate file paths in the <code class=" language-php">phpunit<span class="token punctuation">.</span>xml</code> configuration file, in the <code class=" language-php"><span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>testsuite</span><span class="token punctuation">&gt;</span></span></span></code> <code class=" language-php"><span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>directory</span><span class="token punctuation">&gt;</span></span></span></code> section: the same file is matched by multiple entries.</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token operator">*</span> PHPUnit\<span class="token package">Event<span class="token punctuation">\</span>Test<span class="token punctuation">\</span>PhpunitErrorTriggered</span><span class="token punctuation">:</span> The data provider specified <span class="token keyword keyword-for">for</span> <span class="token scope">Drupal<span class="token punctuation">\</span>Tests<span class="token punctuation">\</span>Composer<span class="token punctuation">\</span>Generator<span class="token punctuation">\</span>BuilderTest<span class="token punctuation">::</span></span>testBuilder is invalid
<span class="token keyword keyword-Class">Class</span> <span class="token string">"Drupal\Composer\Composer"</span> not found
<span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>scheduler<span class="token operator">/</span>web<span class="token operator">/</span>core<span class="token operator">/</span>lib<span class="token operator">/</span>Drupal<span class="token operator">/</span>Core<span class="token operator">/</span>Test<span class="token operator">/</span>PhpUnitTestDiscovery<span class="token punctuation">.</span>php<span class="token punctuation">:</span><span class="token number">147</span>
<span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>scheduler<span class="token operator">/</span>web<span class="token operator">/</span>core<span class="token operator">/</span>scripts<span class="token operator">/</span>run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh<span class="token punctuation">:</span><span class="token number">1050</span>
<span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>scheduler<span class="token operator">/</span>web<span class="token operator">/</span>core<span class="token operator">/</span>scripts<span class="token operator">/</span>run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh<span class="token punctuation">:</span><span class="token number">189</span>

<span class="token operator">*</span> PHPUnit\<span class="token package">Event<span class="token punctuation">\</span>TestRunner<span class="token punctuation">\</span>WarningTriggered</span><span class="token punctuation">:</span> No tests found in <span class="token keyword keyword-class">class</span> <span class="token string">"Drupal\Tests\Composer\Generator\BuilderTest"</span><span class="token punctuation">.</span></code></pre><p>
This means that a class imported by the test class via a <code class=" language-php"><span class="token keyword keyword-use">use</span></code> statement could not be found, and therefore the test was discarded from the discovered list. Likely an autoloading or symlinking issue.</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token operator">*</span>PHPUnit\<span class="token package">Event<span class="token punctuation">\</span>Test<span class="token punctuation">\</span>PhpunitErrorTriggered</span><span class="token punctuation">:</span> The data provider specified <span class="token keyword keyword-for">for</span> <span class="token scope">Drupal<span class="token punctuation">\</span>Tests<span class="token punctuation">\</span>address<span class="token punctuation">\</span>Unit<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Validation<span class="token punctuation">\</span>Constraint<span class="token punctuation">\</span>CountryConstraintValidatorTest<span class="token punctuation">::</span></span>testValidate is invalid
Data Provider method <span class="token scope">Drupal<span class="token punctuation">\</span>Tests<span class="token punctuation">\</span>address<span class="token punctuation">\</span>Unit<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Validation<span class="token punctuation">\</span>Constraint<span class="token punctuation">\</span>CountryConstraintValidatorTest<span class="token punctuation">::</span></span><span class="token function">providerTestValidate</span><span class="token punctuation">(</span><span class="token punctuation">)</span> is not <span class="token keyword keyword-static">static</span>
<span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>scheduler<span class="token operator">/</span>web<span class="token operator">/</span>core<span class="token operator">/</span>lib<span class="token operator">/</span>Drupal<span class="token operator">/</span>Core<span class="token operator">/</span>Test<span class="token operator">/</span>PhpUnitTestDiscovery<span class="token punctuation">.</span>php<span class="token punctuation">:</span><span class="token number">147</span>
<span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>scheduler<span class="token operator">/</span>web<span class="token operator">/</span>core<span class="token operator">/</span>scripts<span class="token operator">/</span>run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh<span class="token punctuation">:</span><span class="token number">1050</span>
<span class="token operator">/</span>builds<span class="token operator">/</span>project<span class="token operator">/</span>scheduler<span class="token operator">/</span>web<span class="token operator">/</span>core<span class="token operator">/</span>scripts<span class="token operator">/</span>run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh<span class="token punctuation">:</span><span class="token number">189</span></code></pre><p>
This means that during test discovery, a data provider that was not specifying the <code class=" language-php"><span class="token keyword keyword-static">static</span></code> modifier was encountered. While this will be a big bold failure if we were in the test execution phase, during discovery it will just be discarded from the test list. The obvious fix here is to add the <code class=" language-php"><span class="token keyword keyword-static">static</span></code> identifier to the data provider method.</p>

---

## [JSON:API's handling of reference fields was changed to support referencing by UUID or revision ID](https://www.drupal.org/node/3519887)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Before 11.2 when updating an entity with a reference field via JSON:API the target entity would be set to the main property of the reference field, which generally is <code class=" language-php">target_id</code>. To support, for example, the use case of <a href="/project/entity_reference_uuid" rel="nofollow">Entity Reference UUID</a> this was changed to instead set the (computed) <code class=" language-php">entity</code> property. Having this (computed) property was already a requirement for reference fields so the only added assumption is that setting that property leads to the correct value being synchronized to the <code class=" language-php">target_id</code> property (or the <code class=" language-php">target_uuid</code> property in the case of Entity Reference UUID).</p>
<p>So to work with JSON:API any custom entity reference implementation must ensure that doing <code class=" language-php"><span class="token variable">$item</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">entity</span> <span class="token operator">=</span> <span class="token variable">$target_entity</span><span class="token punctuation">;</span></code> leads to the correct value being stored in the non-computed property (or properties). This also improves the developer experience for usage outside of JSON:API.</p>

---

## [Tests with PHPUnit 10 attributes are now supported](https://www.drupal.org/node/3447698)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Prior to PHPUnit 10, annotations in special PHP comments, so-called “DocBlocks” or “doc-comments”, were the only means of attaching metadata to code units.</p>
<p>PHPUnit 10 introduced PHP 8 attributes as a replacement for annotations. Annotations are deprecated in PHUnit 11 and will no longer be supported in PHPUnit 12.</p>
<p>Drupal testing framework executed via <code class=" language-php">run<span class="token operator">-</span>tests<span class="token punctuation">.</span>sh</code> now supports executing tests that use attributes in place of annotations.</p>
<p><strong>Mixing attributes and annotations in the same test class or method</strong></p>
<p>Note that per <a href="https://docs.phpunit.de/en/10.5/annotations.html" rel="nofollow">https://docs.phpunit.de/en/10.5/annotations.html</a>,</p>
<blockquote><p>PHPUnit will first look for metadata in attributes before it looks for annotations in comments. When metadata is found in attributes, metadata in comments is ignored.</p></blockquote>
<p>which means that mixing attributes and annotations in the same test class or method needs to be done carefully - there's a risk that a partial attributes implementation shadows a more complete annotations one, with unexpected side effects.</p>
<p><strong>How to migrate tests</strong></p>
<p>The syntax and semantics of test attributes are those defined by the PHPUnit project and documented in <a href="https://docs.phpunit.de/en/10.5/attributes.html" rel="nofollow">this page</a> for PHPUnit 10.5, or the equivalent page in later PHPUnit versions.</p>
<p>The most complete guide on how to convert tests from using annotations to use attributes instead is outlined in this GitHub issue: <a href="https://github.com/sebastianbergmann/phpunit/issues/4502" rel="nofollow">Support PHP 8 attributes for adding metadata to test classes and test methods as well as tested code units</a>.</p>
<p>The following Drupal-specific conventions need to be followed, too:</p>
<ul>
<li>The <code class=" language-php">@group legacy</code> annotation, that in old tests was used to indicate to the Symfony PHPUnit-bridge tests that were expected to check deprecations, must be replaced by the <code class=" language-php"><span class="token shell-comment comment">#[IgnoreDeprecations]</span></code> attribute.</li>
<li>The <code class=" language-php">@covers</code> annotation on the method level has no correspondence to any attribute in PHPUnit 10. There was some discussion in the PHPUnit project to reintroduce it (<a href="https://github.com/sebastianbergmann/phpunit/issues/5558" rel="nofollow">here</a> and <a href="https://github.com/sebastianbergmann/phpunit/issues/5175" rel="nofollow">here</a>) but so far (May 2025) this is off the table. These annotations should be removed from tests, or in case it's relevant to keep memory of the method coverage, it suggested the annotation be changed to <code class=" language-php">@legacy<span class="token operator">-</span>covers</code>.</li>
</ul>
<p><strong>Old style tests:<br>
</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token delimiter">&lt;?php</span>

<span class="token keyword keyword-declare">declare</span><span class="token punctuation">(</span>strict_types<span class="token operator">=</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>KernelTests<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Archiver</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Archiver<span class="token punctuation">\</span>Tar</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * @coversDefaultClass \Drupal\Core\Archiver\Tar
 * @group tar
 */</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">TarTest</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">ArchiverTestBase</span> <span class="token punctuation">{</span>
  
  <span class="token comment" spellcheck="true">/**
   * @covers ::foo()
   */</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">testFoo</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> bool <span class="token punctuation">{</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
  <span class="token punctuation">}</span>
  
  <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span>
</code></pre><p>
<strong>New style tests:<br>
</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token delimiter">&lt;?php</span>

<span class="token keyword keyword-declare">declare</span><span class="token punctuation">(</span>strict_types<span class="token operator">=</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>KernelTests<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Archiver</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Archiver<span class="token punctuation">\</span>Tar</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">PHPUnit<span class="token punctuation">\</span>Framework<span class="token punctuation">\</span>Attributes<span class="token punctuation">\</span>CoversClass</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">PHPUnit<span class="token punctuation">\</span>Framework<span class="token punctuation">\</span>Attributes<span class="token punctuation">\</span>Group</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * Tests Drupal\Core\Archiver\Tar.
 */</span>
<span class="token shell-comment comment">#[CoversClass(Tar::class)]</span>
<span class="token shell-comment comment">#[Group(</span><span class="token string">'tar'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">TarTest</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">ArchiverTestBase</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">/**
   * @legacy-covers ::foo()
   */</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">testFoo</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> bool <span class="token punctuation">{</span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
  <span class="token punctuation">}</span>
  
  <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [Cron service is no longer lazy and proxy class is removed](https://www.drupal.org/node/3484001)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php">cron</code> service is no longer marked <code class=" language-php">lazy</code>, and its proxy <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>ProxyClass<span class="token punctuation">\</span>Cron</span></code> has been removed. Instead, if <code class=" language-php">cron</code> needs to be injected into a service, it is recommended to use service closures instead.</p>
<p>If existing code injecting the cron service is not updated, there may be a slight performance hit due to the different ways that lazy services/proxy classes and service closures optimize for late instantiation (generally 1ms or less). Otherwise, existing code should continue to function as usual.</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
  <span class="token keyword keyword-protected">protected</span> CronInterface <span class="token variable">$cron</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
</code></pre><p>
Service definition:</p>
<pre class="codeblock language-php"><code class=" language-php">services<span class="token punctuation">:</span>
  my_service<span class="token punctuation">:</span>
    <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> Foo\<span class="token package">Bar</span>
    arguments<span class="token punctuation">:</span> <span class="token punctuation">[</span><span class="token string">'@cron'</span><span class="token punctuation">]</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
  <span class="token keyword keyword-protected">protected</span> \<span class="token package">Closure</span> <span class="token variable">$cronClosure</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>

<span class="token keyword keyword-protected">protected</span> <span class="token keyword keyword-function">function</span> <span class="token function">getCron</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> CronInterface <span class="token punctuation">{</span>
  <span class="token keyword keyword-return">return</span> <span class="token punctuation">(</span><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">cronClosure</span><span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><p>
Service definition:</p>
<pre class="codeblock language-php"><code class=" language-php">services<span class="token punctuation">:</span>
  my_service<span class="token punctuation">:</span>
    <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> Foo\<span class="token package">Bar</span>
    arguments<span class="token punctuation">:</span> <span class="token punctuation">[</span><span class="token operator">!</span>service_closure <span class="token string">'@cron'</span><span class="token punctuation">]</span>
</code></pre><p>
Service closures can also be autowired.<br>
<strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
  <span class="token keyword keyword-protected">protected</span> CronInterface <span class="token variable">$cron</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
</code></pre><p>
Service definition:</p>
<pre class="codeblock language-php"><code class=" language-php">services<span class="token punctuation">:</span>
  my_service<span class="token punctuation">:</span>
    <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> Foo\<span class="token package">Bar</span>
    autowire<span class="token punctuation">:</span> <span class="token boolean">true</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Symfony<span class="token punctuation">\</span>Component<span class="token punctuation">\</span>DependencyInjection<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>AutowireServiceClosure</span><span class="token punctuation">;</span>

<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
  <span class="token shell-comment comment">#[AutowireServiceClosure(</span><span class="token string">'cron'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-protected">protected</span> \<span class="token package">Closure</span> <span class="token variable">$cronClosure</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>

<span class="token keyword keyword-protected">protected</span> <span class="token keyword keyword-function">function</span> <span class="token function">getCron</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> CronInterface <span class="token punctuation">{</span>
  <span class="token keyword keyword-return">return</span> <span class="token punctuation">(</span><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">cronClosure</span><span class="token punctuation">)</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [Single-Directory Components properties with enum can define meta:enum with meta-information](https://www.drupal.org/node/3519574)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>Single-Directory Components allow to specify enum properties, which restricts the possible values in the element to our list.</p>
<p>But all page builders, like Experience Builder, that want to use Single-Directory Components require a way to document (and even translate) these values to more human-friendly labels.</p>
<p>The meta:enum attribute will make that feasible, as in the following <code class=" language-php"><span class="token operator">*</span><span class="token punctuation">.</span>component<span class="token punctuation">.</span>yml</code> fragment example:</p>
<pre class="codeblock language-php"><code class=" language-php">    html_tag<span class="token punctuation">:</span>
      type<span class="token punctuation">:</span> string
      title<span class="token punctuation">:</span> <span class="token constant">HTML</span> tag
      enum<span class="token punctuation">:</span>
        <span class="token operator">-</span> a
        <span class="token operator">-</span> button
        <span class="token operator">-</span> span
      meta<span class="token punctuation">:</span>enum<span class="token punctuation">:</span>
        a<span class="token punctuation">:</span> Link
        button<span class="token punctuation">:</span> Button
        span<span class="token punctuation">:</span> Inline
      x<span class="token operator">-</span>translation<span class="token operator">-</span>context<span class="token punctuation">:</span> <span class="token constant">HTML</span> tag
      <span class="token keyword keyword-default">default</span><span class="token punctuation">:</span> button
</code></pre><p>
If your <code class=" language-php">meta<span class="token punctuation">:</span>enum</code> doesn't define some valid value from the <code class=" language-php">enum</code> list of values the raw enum values will be used instead.</p>

---

## [Workspaces no longer creates a "Stage" workspace on installation](https://www.drupal.org/node/3527964)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Workspaces no longer creates a default 'Stage' workspace when it is installed.</p>
<p>This will not affect existing sites, but it could affect contrib modules or install profiles that rely implicitly on the workspace existing.</p>

---

## [Block content access classes moved from block_content to core](https://www.drupal.org/node/3527501)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The following were moved from the block_content module to core, with corresponding namespace changes:<br>
<code class=" language-php">Drupal\<span class="token package">block_content<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>AccessGroupAnd</span></code><br>
<code class=" language-php">Drupal\<span class="token package">block_content<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>DependentAccessInterface</span></code><br>
<code class=" language-php">Drupal\<span class="token package">block_content<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>RefinableDependentAccessInterface</span></code><br>
<code class=" language-php">Drupal\<span class="token package">block_content<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>RefinableDependentAccessTrait</span></code></p>
<p>Updated fully qualified names:<br>
<code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>AccessGroupAnd</span></code><br>
<code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>DependentAccessInterface</span></code><br>
<code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>RefinableDependentAccessInterface</span></code><br>
<code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Access<span class="token punctuation">\</span>RefinableDependentAccessTrait</span></code></p>

---

## [drupal_requirements_severity() and REQUIREMENT_* severity constants have been deprecated.](https://www.drupal.org/node/3410939)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<h2>Requirements severity constants have been replaced with an enum</h2>
<table>
<tbody><tr>
<th>Before</th>
<th>After</th>
</tr>
<tr>
<td><code class=" language-php"><span class="token constant">REQUIREMENT_INFO</span></code></td>
<td><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>Requirement<span class="token punctuation">\</span>RequirementSeverity<span class="token punctuation">::</span></span>Info</code></td>
</tr>
<tr>
<td><code class=" language-php"><span class="token constant">REQUIREMENT_OK</span></code></td>
<td><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>Requirement<span class="token punctuation">\</span>RequirementSeverity<span class="token punctuation">::</span></span><span class="token constant">OK</span></code></td>
</tr>
<tr>
<td><code class=" language-php"><span class="token constant">REQUIREMENT_WARNING</span></code></td>
<td><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>Requirement<span class="token punctuation">\</span>RequirementSeverity<span class="token punctuation">::</span></span>Warning</code></td>
</tr>
<tr>
<td><code class=" language-php"><span class="token constant">REQUIREMENT_ERROR</span></code></td>
<td><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>Requirement<span class="token punctuation">\</span>RequirementSeverity<span class="token punctuation">::</span></span>Error</code></td>
</tr>
</tbody></table>
<h2>Deprecated functions related to requirements severity</h2>
<p><code class=" language-php"><span class="token function">drupal_requirements_severity</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> has been deprecated and has been replaced with a method on the enum:  <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>Requirement<span class="token punctuation">\</span>RequirementSeverity<span class="token punctuation">::</span></span><span class="token function">getMaxSeverity</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></p>
<p><code class=" language-php"><span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Render<span class="token punctuation">\</span>Element<span class="token punctuation">\</span>StatusReport<span class="token punctuation">::</span></span><span class="token function">getSeverities</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> has been deprecated with no replacement. Using the enum directly is a better approach.</p>

---

## [Drupal\node\NodePermissions now requires EntityTypeManagerInterface in its constructor](https://www.drupal.org/node/3526030)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core

### Description

<p><code class=" language-php"><span class="token scope">Drupal<span class="token punctuation">\</span>node<span class="token punctuation">\</span>NodePermissions<span class="token punctuation">::</span></span><span class="token function">construct</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> now requires a single <code class=" language-php">EntityTypeManagerInterface</code> argument. This class is @internal so the change should not affect contrib or custom code.</p>

---

## [Calling \Drupal\Core\Extension\ThemeInstaller::__construct() without the $componentPluginManager argument is deprecated](https://www.drupal.org/node/3525649)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Calling \Drupal\Core\Extension\ThemeInstaller::__construct() without the $componentPluginManager argument is deprecated in drupal:11.2.0 and it will be required in drupal:12.0.0.</p>

---

## [Blocks are no longer created automatically when themes or modules are enabled during config sync](https://www.drupal.org/node/3525560)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p><code class=" language-php">block_themes_installed</code> and <code class=" language-php">block_modules_installed</code> no longer execute when configuration is syncing.</p>
<p>During recipe application. Blocks will no longer be duplicated from the existing theme, so recipe authors that change themes will need to be declarative about which blocks they want to import and place on the site.</p>
<p>When enabling themes through the UI or with Drush there is no change.</p>

---

## [Package Manager can allow Composer operations directly on the live site in some situations](https://www.drupal.org/node/3506770)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Package manager can now, in some circumstances, operate directly on the live site instead of a sandbox. For example, if you have a site on your local machine, of which you are the only user, and you want to use Project Browser to add modules.</p>
<p>Previously, Package Manager would <em>always</em> create a separate, sandboxed copy of the Drupal site, run Composer on it, and then copy the changes back. This minimizes risk to the live site, but can also lead to poor performance. This is a reasonable trade-off because keeping the Drupal site unbroken is Package Manager's main concern.</p>
<p>To operate directly on the live site, two conditions must be met.</p>
<ol>
<li>A new setting, <code class=" language-php">package_manager_allow_direct_write</code>, must be set to TRUE.</li>
<li>Individual subclasses of <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>package_manager<span class="token punctuation">\</span>SandboxManagerBase</span></code> must have the <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>package_manager<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>AllowDirectWrite</span></code> attribute applied to them. For example:</li>
</ol>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>package_manager<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>AllowDirectWrite</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>package_manager<span class="token punctuation">\</span>SandboxManagerBase</span><span class="token punctuation">;</span>

<span class="token shell-comment comment">#[AllowDirectWrite]</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">MyStage</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">SandboxManagerBase</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">// ...magic time</span>

<span class="token punctuation">}</span>
</code></pre><p>
Classes that extend MyStage will <em>not</em> inherit the attribute.</p>
<p>This is a reasonable balance between safety and flexibility. Package Manager will continue to prioritize safety.  But developers who understand the intricacies of their specific use cases will be able to opt out of sandboxing.</p>
<p>If in doubt, <em>don't</em> add <code class=" language-php">AllowDirectWrite</code> to your SandboxManagerBase subclass; only add it if you know exactly what you're doing, and why.</p>

---

## [The Syndicate block is deprecated](https://www.drupal.org/node/3519248)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>node<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Block<span class="token punctuation">\</span>SyndicateBlock</span></code> is deprecated. No replacement is provided.</p>

---

## [The 'cachetags' database table is now purged during cache rebuild](https://www.drupal.org/node/3522219)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The 'cachetags' database table is now purged during cache rebuild. This happens within a call to drupal_flush_all_caches(), immediately after the cache bins are emptied.</p>
<p>This is done by having the <code class=" language-php">cache_tags<span class="token punctuation">.</span>invalidator</code> and <code class=" language-php">cache_tags<span class="token punctuation">.</span>invalidator<span class="token punctuation">.</span>checksum</code> services implement a new interface <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Cache<span class="token punctuation">\</span>CacheTagsPurgeInterface</span></code> and its <code class=" language-php"><span class="token function">purge</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method.</p>
<p>If alternative implementations of the cache tag checksum service also need to explicitly purge their data, they can implement this interface.</p>
<p>Previously, the 'cachetags' table could grow endlessly and could get large if there were a large number of cache tags being invalidated.</p>

---

## [New image effect, Convert to AVIF (with fallback), added](https://www.drupal.org/node/3511540)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>Since <a href="https://www.drupal.org/node/3348348" rel="nofollow">GDToolkit supports AVIF image format</a>, servers that have AVIF support can create AVIF image derivatives. Since we can't guarantee support on all configurations we can't use AVIF as the default format.</p>
<p>A new image conversion effect has been added that converts to AVIF on servers that support it, but allows a fallback image style (such as WEBP) when AVIF is not available.</p>

---

## [Custom keys in $_SESSION are deprecated](https://www.drupal.org/node/3518914)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Custom session values stored directly in the <code class=" language-php"><span class="token global">$_SESSION</span></code> super global are deprecated. Instead access session attributes via <code class=" language-php"><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSession</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">set</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and <code class=" language-php"><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">getSession</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>.</p>
<h3>Deprecated:</h3>
<h4>Set a value in the session</h4>
<pre class="codeblock language-php"><code class=" language-php"><span class="token global">$_SESSION</span><span class="token punctuation">[</span><span class="token string">'test_key'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'test_value'</span><span class="token punctuation">;</span>
</code></pre><h4>Get a value from the session</h4>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$value</span> <span class="token operator">=</span> <span class="token global">$_SESSION</span><span class="token punctuation">[</span><span class="token string">'test_key'</span><span class="token punctuation">]</span> <span class="token operator">?</span><span class="token operator">?</span> <span class="token string">'default_value'</span><span class="token punctuation">;</span>
</code></pre><h3>Recommended:</h3>
<h4>Set a value in the session</h4>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSession</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">set</span><span class="token punctuation">(</span><span class="token string">'test_key'</span><span class="token punctuation">,</span> <span class="token string">'test_value'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><h4>Get a value from the session</h4>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$value</span> <span class="token operator">=</span> <span class="token variable">$request</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSession</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'test_key'</span><span class="token punctuation">,</span> <span class="token string">'default_value'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [system/base split into smaller, conditionally loaded libraries](https://www.drupal.org/node/3523938)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Various CSS files previously included in the system/base library have been split into smaller asset libraries, and are now conditionally loaded with the elements or templates that require them.</p>
<p>Themes or modules relying on CSS specified in those libraries may need to include the new library in #attached.</p>
<p>Themes that were overriding the files may need to adjust to override the new libraries.</p>
<p>See <a href="https://www.drupal.org/node/3432346" rel="nofollow">https://www.drupal.org/node/3432346</a> for examples of the changes needed.</p>

---

## [Contextual links now use native JavaScript instead of BackboneJS](https://www.drupal.org/node/3523287)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<table>
<caption>
<h2>New API</h2>
</caption>
<tbody><tr>
<th>Drupal Event</th>
<th>Drupal.contextual</th>
<th>Drupal.contextualToolbar</th>
</tr>
<tr>
<td>
<pre class="codeblock language-php"><code class=" language-php">drupalContextualLinkAdded
  <span class="token variable">$el</span> <span class="token operator">=</span><span class="token operator">&gt;</span> jQuery
  <span class="token variable">$region</span> <span class="token operator">=</span><span class="token operator">&gt;</span> jQuery
  contextualModelView
</code></pre></td>
<td>
<pre class="codeblock language-php"><code class=" language-php">instances<span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> of ContextualModelView
ContextualModelView
  <span class="token variable">$contextual</span>
  <span class="token variable">$region</span>
  modelId
  regionIsHovered
  strings
  timer
  title
  hasFocus
  isLocked
  isOpen


</code></pre></td>
<td>
<pre class="codeblock language-php"><code class=" language-php">model<span class="token punctuation">:</span> instance of ContextualToolbarModelView
ContextualToolbarModelView
  <span class="token variable">$el</span>
  strings
  tabbingContext
  isViewing

</code></pre></td>
</tr>
</tbody></table>
<table>
<caption>
<h2>Old API</h2>
</caption>
<tbody><tr>
<th>Drupal Event</th>
<th>Drupal.contextual</th>
<th>Drupal.contextualToolbar</th>
</tr>
<tr>
<td>
<pre class="codeblock language-php"><code class=" language-php">drupalContextualLinkAdded
  <span class="token variable">$el</span> <span class="token operator">=</span><span class="token operator">&gt;</span> jQuery
  <span class="token variable">$region</span> <span class="token operator">=</span><span class="token operator">&gt;</span> jQuery
  model <span class="token operator">=</span><span class="token operator">&gt;</span> StateModel</code></pre></td>
<td>
<pre class="codeblock language-php"><code class=" language-php">AuralView
KeyboardView
RegionView
VisualView
StateModel <span class="token punctuation">(</span>Backbone model <span class="token operator">+</span><span class="token punctuation">)</span><span class="token punctuation">:</span>
  title
  regionIsHovered
  hasFocus
  isOpen
  isLocked
  <span class="token function">toggleOpen</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token function">close</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token function">focus</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token function">blur</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  
collection<span class="token punctuation">:</span> Backbone collection of
  models<span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> of StateModel
regionViews<span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> of RegionView
views<span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> of objects<span class="token punctuation">:</span>
  aural <span class="token operator">=</span><span class="token operator">&gt;</span> AuralView
  keyboard <span class="token operator">=</span><span class="token operator">&gt;</span> KeyboardView
  visual <span class="token operator">=</span><span class="token operator">&gt;</span> VisualView

events bound on backbone models<span class="token punctuation">:</span>
  change <span class="token punctuation">(</span>from the <span class="token operator">*</span>Views models<span class="token punctuation">,</span> triggers a renrender<span class="token punctuation">)</span>
  change<span class="token punctuation">:</span>hasFocus <span class="token punctuation">(</span>from the regionView model<span class="token punctuation">)</span>
  
</code></pre></td>
<td>
<pre class="codeblock language-php"><code class=" language-php">AuralView <span class="token punctuation">(</span>not exposed<span class="token punctuation">)</span>
VisualView <span class="token punctuation">(</span>not exposed<span class="token punctuation">)</span> 
StateModel <span class="token punctuation">(</span>Backbone model <span class="token operator">+</span><span class="token punctuation">)</span><span class="token punctuation">:</span>
  isViewing
  isVisible
  contextualCount
  tabbingContext
  <span class="token function">countContextualLinks</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token function">lockNewContextualLinks</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  <span class="token function">updateVisibility</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
  
model <span class="token operator">=</span><span class="token operator">&gt;</span> StateModel

events bound on model<span class="token punctuation">:</span>
  change
  change<span class="token punctuation">:</span>contextualCount
  change<span class="token punctuation">:</span>isViewing

events bound on collection<span class="token punctuation">:</span>
  add
  remove
  reset
</code></pre></td>
</tr>
<tr>
<th>Backbone View</th>
<th>Backbone Model</th>
<th>Backbone Collection</th>
</tr>
<tr>
<td>
<pre class="codeblock language-php"><code class=" language-php">el
cid
<span class="token variable">$el</span>
prototype<span class="token punctuation">:</span>
  $
  bind
  delegate
  delegateEvents
  initialize
  listenTo
  listenToOnce
  off
  on
  once
  preinitialize
  remove
  render
  setElement
  stopListening
  tagName
  trigger
  unbind
  undelegate
  undelegateEvents</code></pre></td>
<td>
<pre class="codeblock language-php"><code class=" language-php">cid
attributes
changed
prototype<span class="token punctuation">:</span>
  bind
  chain
  changed
  changedAttributes
  cidPrefix
  clear
  clone
  destroy
  escape
  fetch
  get
  has
  hasChanged
  idAttribute
  initialize
  invert
  isEmpty
  isNew
  isValid
  keys
  listenTo
  listenToOnce
  matches
  off
  omit
  on
  once
  pairs
  parse
  pick
  preinitialize
  previous
  previousAttributes
  save
  set
  stopListening
  sync
  toJSON
  trigger
  unbind
  unset
  url
  validationError
  values</code></pre></td>
<td>
<pre class="codeblock language-php"><code class=" language-php">length
models
prototype<span class="token punctuation">:</span> 
  add
  all
  any
  at
  bind
  chain
  clone
  collect
  contains
  countBy
  create
  detect
  difference
  drop
  each
  entries
  every
  fetch
  filter
  find
  findIndex
  findLastIndex
  findWhere
  first
  foldl
  foldr
  <span class="token keyword keyword-forEach">forEach</span>
  get
  groupBy
  has
  head
  <span class="token keyword keyword-include">include</span>
  includes
  indexBy
  indexOf
  initial
  initialize
  inject
  invoke
  isEmpty
  keys
  last
  lastIndexOf
  listenTo
  listenToOnce
  map
  max
  min
  model
  modelId
  off
  on
  once
  parse
  partition
  pluck
  pop
  preinitialize
  push
  reduce
  reduceRight
  reject
  remove
  reset
  rest
  sample
  select
  set
  shift
  shuffle
  size
  slice
  some
  sort
  sortBy
  stopListening
  sync
  tail
  take
  toArray
  toJSON
  trigger
  unbind
  unshift
  values
  where
  without</code></pre></td>
</tr>
</tbody></table>

---

## [Calling \Drupal\Core\Menu\MenuActiveTrail::__construct() without the $pathMatcher argument is deprecated](https://www.drupal.org/node/3523495)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Calling <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Menu<span class="token punctuation">\</span>MenuActiveTrail<span class="token punctuation">::</span></span><span class="token function">__construct</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> without the <code class=" language-php"><span class="token variable">$pathMatcher</span></code> argument is deprecated in drupal:11.2.0 and it will be required in drupal:12.0.0.</p>

---

## [Renderer::render() $is_root_call parameter deprecated](https://www.drupal.org/node/3497318)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Use Renderer::renderRoot() instead.</p>

---

## [Views blocks' `items_per_page` setting can no longer be `none`](https://www.drupal.org/node/3522240)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Views blocks always have an <code class=" language-php">items_per_page</code> setting -- that is, the number of items from the view that should be visible in the block.</p>
<p>Until <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3520946" title="Status: Closed (fixed)">#3520946: Make View blocks (block.settings.view_block:*) fully validatable</a></span>, that setting could either be a number, or <code class=" language-php">none</code> to defer to whatever the view itself has configured. Now, <code class=" language-php">none</code> is no longer supported; to defer to the view, <code class=" language-php">items_per_page</code> should be null.</p>

---

## [New Recipe Unpack composer plugin](https://www.drupal.org/node/3522189)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The Recipe Unpacking system is a Composer plugin that manages "drupal-recipe" packages. Recipes are special Composer packages designed to bootstrap Drupal projects with necessary dependencies. When a recipe is required, this plugin "unpacks" it by moving the recipe's dependencies directly into your project's root <code class=" language-php">composer<span class="token punctuation">.</span>json</code>, and removes the recipe as a project dependency.</p>
<h2>What is Unpacking?</h2>
<p>Unpacking is the process where:</p>
<ol>
<li>A recipe's dependencies are added to your project's root <code class=" language-php">composer<span class="token punctuation">.</span>json</code></li>
<li>The recipe itself is removed from your dependencies</li>
<li>The <code class=" language-php">composer<span class="token punctuation">.</span>lock</code> and vendor installation files are updated accordingly</li>
<li>The recipe will remain in the project's recipes folder so it can be applied</li>
</ol>
<h2>Commands</h2>
<h3><code class=" language-php">drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack</code></h3>
<p>Unpack a recipe package that's already required in your project.</p>
<pre class="codeblock language-php"><code class=" language-php">composer drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack drupal<span class="token operator">/</span>example_recipe
</code></pre><p>
Unpack all recipes that are required in your project.</p>
<pre class="codeblock language-php"><code class=" language-php">composer drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack
</code></pre><h4>Options</h4>
<p>This command doesn't take additional options.</p>
<h2>Automatic Unpacking</h2>
<h3>After <code class=" language-php">composer <span class="token keyword keyword-require">require</span></code></h3>
<p>By default, recipes are automatically unpacked after running <code class=" language-php">composer <span class="token keyword keyword-require">require</span></code> for a recipe package:</p>
<pre class="codeblock language-php"><code class=" language-php">composer <span class="token keyword keyword-require">require</span> drupal<span class="token operator">/</span>example_recipe
</code></pre><p>
This will:</p>
<ol>
<li>Download the recipe and its dependencies</li>
<li>Add the recipe's dependencies to your project's root <code class=" language-php">composer<span class="token punctuation">.</span>json</code></li>
<li>Remove the recipe itself from your dependencies</li>
<li>Update your <code class=" language-php">composer<span class="token punctuation">.</span>lock</code> file</li>
</ol>
<h3>After <code class=" language-php">composer create<span class="token operator">-</span>project</code></h3>
<p>Recipes are always automatically unpacked when creating a new project from a template that requires this plugin:</p>
<pre class="codeblock language-php"><code class=" language-php">composer create<span class="token operator">-</span>project drupal<span class="token operator">/</span>recommended<span class="token operator">-</span>project my<span class="token operator">-</span>project
</code></pre><p>
Any recipes included in the project template will be unpacked during installation, as long as the plugin is enabled.</p>
<h2>Add recipe unpacking to an existing site</h2>
<pre class="codeblock language-php"><code class=" language-php">composer <span class="token keyword keyword-require">require</span> drupal<span class="token operator">/</span>core<span class="token operator">-</span>recipe<span class="token operator">-</span>unpack
</code></pre><p>
Ensure the plugin is in the list of allowed plugins:</p>
<pre class="codeblock language-php"><code class=" language-php">composer config allow<span class="token operator">-</span>plugins<span class="token punctuation">.</span>drupal<span class="token operator">/</span>core<span class="token operator">-</span>recipe<span class="token operator">-</span>unpack <span class="token boolean">true</span>
</code></pre><p>
Once this is done you will be able to unpack existing recipes using the <code class=" language-php">drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack</code> command.</p>
<p>The unpack plugin can be added to projects that are using Drupal 11.1 or older; it does not, itself, require any particular version of Drupal, although recipes have only been supported in Drupal since 10.3.</p>
<h2>Configuration</h2>
<p>Configuration options are set in the <code class=" language-php">extra</code> section of your <code class=" language-php">composer<span class="token punctuation">.</span>json</code> file:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token punctuation">{</span>
  <span class="token string">"extra"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
    <span class="token string">"drupal-recipe-unpack"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
      <span class="token string">"ignore"</span><span class="token punctuation">:</span> <span class="token punctuation">[</span><span class="token string">"drupal/recipe_to_ignore"</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token string">"on-require"</span><span class="token punctuation">:</span> <span class="token boolean">true</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre><h3>Available Options</h3>
<ul>
<li><code class=" language-php">ignore</code> - an array - defaults to <code class=" language-php"><span class="token punctuation">[</span><span class="token punctuation">]</span></code>. List of recipe packages to exclude from unpacking.</li>
<li><code class=" language-php">on<span class="token operator">-</span><span class="token keyword keyword-require">require</span></code> - a boolean - defaults to <code class=" language-php"><span class="token boolean">true</span></code>. Whether to automatically unpack recipes when required.</li>
</ul>
<h2>How Recipe Unpacking Works</h2>
<ol>
<li>The system identifies packages of type <code class=" language-php">drupal<span class="token operator">-</span>recipe</code> during installation</li>
<li>For each recipe not in the ignore list, it:
<ul>
<li>Extracts its dependencies</li>
<li>Adds them to the root <code class=" language-php">composer<span class="token punctuation">.</span>json</code></li>
<li>Recursively processes any dependencies that are also recipes</li>
<li>Removes the recipe and any dependencies that are also recipes from the root <code class=" language-php">composer<span class="token punctuation">.</span>json</code></li>
<p>  
</p></ul></li>
<li>Updates all necessary Composer files:
<ul>
<li><code class=" language-php">composer<span class="token punctuation">.</span>json</code></li>
<li><code class=" language-php">composer<span class="token punctuation">.</span>lock</code></li>
<li><code class=" language-php">vendor<span class="token operator">/</span>composer<span class="token operator">/</span>installed<span class="token punctuation">.</span>json</code></li>
<li><code class=" language-php">vendor<span class="token operator">/</span>composer<span class="token operator">/</span>installed<span class="token punctuation">.</span>php</code></li>
</ul>
</li>
</ol>
<h2>Cases Where Recipes Will Not Be Unpacked</h2>
<p>Recipes will <strong>not</strong> be unpacked in the following scenarios:</p>
<p>1. <strong>Explicit Ignore List</strong>: If the recipe is listed in the <code class=" language-php">ignore</code> array in your <code class=" language-php">extra<span class="token punctuation">.</span>drupal<span class="token operator">-</span>recipe<span class="token operator">-</span>unpack</code> configuration</p>
<pre class="codeblock language-php"><code class=" language-php">   <span class="token punctuation">{</span>
     <span class="token string">"extra"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
       <span class="token string">"drupal-recipe-unpack"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
         <span class="token string">"ignore"</span><span class="token punctuation">:</span> <span class="token punctuation">[</span><span class="token string">"drupal/recipe_name"</span><span class="token punctuation">]</span>
       <span class="token punctuation">}</span>
     <span class="token punctuation">}</span>
   <span class="token punctuation">}</span>
   </code></pre><p>
2. <strong>Disabled Automatic Unpacking</strong>: If <code class=" language-php">on<span class="token operator">-</span><span class="token keyword keyword-require">require</span></code> is set to <code class=" language-php"><span class="token boolean">false</span></code> in your <code class=" language-php">extra<span class="token punctuation">.</span>drupal<span class="token operator">-</span>recipe<span class="token operator">-</span>unpack</code> configuration</p>
<pre class="codeblock language-php"><code class=" language-php">   <span class="token punctuation">{</span>
     <span class="token string">"extra"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
       <span class="token string">"drupal-recipe-unpack"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
         <span class="token string">"on-require"</span><span class="token punctuation">:</span> <span class="token boolean">false</span>
       <span class="token punctuation">}</span>
     <span class="token punctuation">}</span>
   <span class="token punctuation">}</span>
  </code></pre><p>
3. <strong>Development Dependencies</strong>: Recipes in the <code class=" language-php"><span class="token keyword keyword-require">require</span><span class="token operator">-</span>dev</code> section are not automatically unpacked</p>
<pre class="codeblock language-php"><code class=" language-php">   <span class="token punctuation">{</span>
     <span class="token string">"require-dev"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
       <span class="token string">"drupal/dev_recipe"</span><span class="token punctuation">:</span> <span class="token string">"^1.0"</span>
     <span class="token punctuation">}</span>
   <span class="token punctuation">}</span>
   </code></pre><p>   You will need to manually unpack these using the <code class=" language-php">drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack</code> command if desired.</p>
<p>4. <strong>With <code class=" language-php"><span class="token operator">--</span>no<span class="token operator">-</span>install</code> Option</strong>: When using <code class=" language-php">composer <span class="token keyword keyword-require">require</span></code> with the <code class=" language-php"><span class="token operator">--</span>no<span class="token operator">-</span>install</code> flag</p>
<pre class="codeblock language-php"><code class=" language-php">   composer <span class="token keyword keyword-require">require</span> drupal<span class="token operator">/</span>example_recipe <span class="token operator">--</span>no<span class="token operator">-</span>install
   </code></pre><p>   In this case, you'll need to run <code class=" language-php">composer install</code> afterward and then manually unpack using the <code class=" language-php">drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack</code> command.</p>
<h2>Example Usage Scenarios</h2>
<h3>Basic Recipe Installation</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token shell-comment comment"># This will automatically install and unpack the recipe</span>
composer <span class="token keyword keyword-require">require</span> drupal<span class="token operator">/</span>example_recipe
</code></pre><p>
The result:</p>
<ul>
<li>Dependencies from <code class=" language-php">drupal<span class="token operator">/</span>example_recipe</code> are added to your root <code class=" language-php">composer<span class="token punctuation">.</span>json</code></li>
<li><code class=" language-php">drupal<span class="token operator">/</span>example_recipe</code> itself is removed from your dependencies</li>
<li>You'll see a message: "drupal/example_recipe unpacked successfully."</li>
</ul>
<h3>Manual Recipe Unpacking</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token shell-comment comment"># First require the recipe without unpacking</span>
composer <span class="token keyword keyword-require">require</span> drupal<span class="token operator">/</span>example_recipe <span class="token operator">--</span>no<span class="token operator">-</span>install
composer install

<span class="token shell-comment comment"># Then manually unpack it</span>
composer drupal<span class="token punctuation">:</span>recipe<span class="token operator">-</span>unpack drupal<span class="token operator">/</span>example_recipe
</code></pre><h3>Creating a New Project with Recipes</h3>
<pre class="codeblock language-php"><code class=" language-php">composer create<span class="token operator">-</span>project drupal<span class="token operator">/</span>recommended<span class="token operator">-</span>project my<span class="token operator">-</span>project
</code></pre><p>
Any recipes included in the project template will be automatically unpacked during installation.</p>
<h2>Best Practices</h2>
<ol>
<li><strong>Review Recipe Contents</strong>: Before requiring a recipe, review its dependencies to understand what will be added to your project.</li>
<li><strong>Consider Versioning</strong>: When a recipe is unpacked, its version constraints for dependencies are merged with your existing constraints, which may result in complex version requirements.</li>
<li><strong>Dev Dependencies</strong>: Be cautious when unpacking development recipes, as their dependencies will be moved to the main <code class=" language-php"><span class="token keyword keyword-require">require</span></code> section, not <code class=" language-php"><span class="token keyword keyword-require">require</span><span class="token operator">-</span>dev</code>.</li>
<li><strong>Custom Recipes</strong>: When creating custom recipes, ensure they have the correct package type <code class=" language-php">drupal<span class="token operator">-</span>recipe</code> and include appropriate dependencies.</li>
</ol>
<h2>Troubleshooting</h2>
<h3>Recipe Not Unpacking</h3>
<ul>
<li>Check if the package type is <code class=" language-php">drupal<span class="token operator">-</span>recipe</code></li>
<li>Verify it's not in your ignore list</li>
<li>Confirm it's not in <code class=" language-php"><span class="token keyword keyword-require">require</span><span class="token operator">-</span>dev</code> (which requires manual unpacking)</li>
<li>Ensure you haven't used the <code class=" language-php"><span class="token operator">--</span>no<span class="token operator">-</span>install</code> flag without following up with installation and manual unpacking</li>
</ul>
<h3>Unpacking Errors</h3>
<p>If you encounter issues during unpacking:</p>
<ul>
<li>Check Composer's error output for specific issues and run with the <code class=" language-php"><span class="token operator">--</span>verbose</code> flag</li>
<li>Verify that version constraints between your existing dependencies and the recipe's dependencies are compatible</li>
<li>For manual troubleshooting, consider temporarily setting <code class=" language-php">on<span class="token operator">-</span><span class="token keyword keyword-require">require</span></code> to <code class=" language-php"><span class="token boolean">false</span></code> and unpacking recipes one by one</li>
</ul>
<h2>Internal Architecture</h2>
<p>The Recipe Unpacking system consists of several key components:</p>
<ul>
<li><code class=" language-php">Plugin</code>: The main Composer plugin that hooks into Composer events</li>
<li><code class=" language-php">UnpackManager</code>: Manages the unpacking process</li>
<li><code class=" language-php">Unpacker</code>: Handles the details of unpacking a specific package</li>
<li><code class=" language-php">UnpackCollection</code>: Tracks packages to be unpacked and their dependencies</li>
<li><code class=" language-php">UnpackOptions</code>: Manages configuration options for the unpacking process</li>
<li><code class=" language-php">RootComposer</code>: Provides access to and manipulation of the root composer files</li>
</ul>

---

## [New #placeholder_strategy_denylist key for render arrays with lazy builders](https://www.drupal.org/node/3511562)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Render arrays that specify a #lazy_builder may now specify a #placeholder_strategy_denylist key at the same level.</p>
<p>This allows placeholders to restrict the strategy that they're replaced by, e.g.</p>
<pre class="codeblock language-php"><code class=" language-php"> <span class="token variable">$build</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
      <span class="token string">'#lazy_builder'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span><span class="token scope"><span class="token keyword keyword-static">static</span><span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span> <span class="token class-name"><span class="token punctuation">.</span></span> <span class="token string">'::renderMessages'</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token variable">$element</span><span class="token punctuation">[</span><span class="token string">'#display'</span><span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token string">'#create_placeholder'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token constant">TRUE</span><span class="token punctuation">,</span>
      <span class="token comment" spellcheck="true">// Prevent this placeholder being handled by big_pipe.</span>
      <span class="token string">'#placeholder_strategy_denylist'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token scope">BigPipeStrategy<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token constant">TRUE</span><span class="token punctuation">,</span>
        <span class="token scope">CachedStrategy<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token constant">TRUE</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>

</code></pre><p>
In core this is used to prevent the messages block, which shows on every page in most configurations and is very cheap to render, from being rendered by big pipe, so that it is possible to skip loading big pipe's JavaScript when it is otherwise the only placeholder on the page. In general, the key can be skipped for most situations and the most appropriate render strategy will be selected.</p>

---

## [New config action to add a component to a Layout](https://www.drupal.org/node/3519491)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>New Config Action for adding new components to config entities using layout builder from recipes.</p>
<p>You can use it like this:</p>
<pre class="codeblock language-php"><code class=" language-php">config<span class="token punctuation">:</span>
  actions<span class="token punctuation">:</span>
    dashboard<span class="token punctuation">.</span>dashboard<span class="token punctuation">.</span>welcome<span class="token punctuation">:</span>
      addComponentToLayout<span class="token punctuation">:</span>
        section<span class="token punctuation">:</span> <span class="token number">0</span>
        position<span class="token punctuation">:</span> <span class="token number">4</span>
        component<span class="token punctuation">:</span>
          region<span class="token punctuation">:</span>
            layout_twocol_section<span class="token punctuation">:</span> <span class="token string">'second'</span>
          default_region<span class="token punctuation">:</span> content
          configuration<span class="token punctuation">:</span>
            id<span class="token punctuation">:</span> dashboard_text_block
            label<span class="token punctuation">:</span> <span class="token string">'My new dashboard with weight from a plugin'</span>
            label_display<span class="token punctuation">:</span> <span class="token string">'visible'</span>
            provider<span class="token punctuation">:</span> <span class="token string">'dashboard'</span>
            context_mapping<span class="token punctuation">:</span> <span class="token punctuation">{</span> <span class="token punctuation">}</span>
            text<span class="token punctuation">:</span>
              value<span class="token punctuation">:</span> '<span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>p</span><span class="token punctuation">&gt;</span></span></span>My <span class="token keyword keyword-new">new</span> <span class="token class-name">dashboard</span> with weight from plugin<span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>p</span><span class="token punctuation">&gt;</span></span></span>'
              format<span class="token punctuation">:</span> <span class="token string">'basic_html'</span>
</code></pre><p>
The name of the action is addComponentToLayout, and its arguments are:</p>
<ul>
<li><strong>section:</strong> The delta of the section where our new component is going to be added</li>
<li><strong>position:</strong> The position at which should be added. Don't confuse this with weight, which will be automatically calculated.</li>
<li><strong>component:</strong> An array containing the component definition.</li>
</ul>
<p>Inside of <code class=" language-php">component</code>, we can define a <strong>region</strong> that maps the potential layout with the region inside the layout to use. You can also use <strong>default_region</strong> in case the layout id doesn't match any of the above mapped layout regions.</p>
<p>It will have a <strong>configuration</strong> key, with whatever schema your plugin will expect. At least an <strong>id</strong> with the <code class=" language-php">plugin_id</code> will be expected.</p>
<p>This action can be used against any config entity using layout builder. E.g. an entity view display from core, or a dashboard from the contrib module <code class=" language-php">dashboard</code>, as in the example.</p>

---

## [SectionListTrait::setSection() is now a public method](https://www.drupal.org/node/3522612)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>layout_builder<span class="token punctuation">\</span>SectionListTrait<span class="token punctuation">::</span></span><span class="token function">setSection</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> is now a public method and will be available on every class that uses <code class=" language-php">SectionListTrait</code>.</p>

---

## [PHPUnit 11 support](https://www.drupal.org/node/3521395)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>PHPUnit 11 can now be used for testing. While the shipped version remains PHPUnit 10, it's possible to update to PHPUnit via <code class=" language-php">composer update phpunit<span class="token operator">/</span>phpunit <span class="token operator">--</span>with<span class="token operator">-</span>dependencies</code>. </p>
<p>Drupal core testing on PHP 8.4 requires PHPUnit 11 as a minimum, and automatically updates the version in the GitLab test runs.</p>
<p>Note that until <span class="project-issue-issue-link project-issue-status-info project-issue-status-13"><a href="https://www.drupal.org/project/drupal/issues/3497431" title="Status: Needs work">#3497431: Deprecate TestDiscovery test file scanning, use PHPUnit API instead</a></span> is committed, using test attributes introduced in PHPUnit 10 (replacing test annotations) is not yet fully supported.</p>

---

## [Additional 'Update Manager' deprecations](https://www.drupal.org/node/3522119)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>In addition to the changes described at <a href="https://www.drupal.org/node/3512364" rel="nofollow">authorize.php, the FileTransfer and Updater systems, and all related code is deprecated</a>, additional services, functions and a theme template are deprecated:</p>
<ol>
<li>The <code class=" language-php">update<span class="token punctuation">.</span>root</code> service and <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>update<span class="token operator">/</span>src<span class="token operator">/</span>UpdateRoot<span class="token punctuation">.</span>php</code></li>
<li><code class=" language-php"><span class="token function">_update_manager_cache_directory</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">_update_manager_extract_directory</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">_update_manager_unique_identifier</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">update_clear_update_disk_cache</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">update_delete_file_if_stale</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li>The <code class=" language-php">authorize<span class="token operator">-</span>report</code> theme template, <code class=" language-php">authorize<span class="token operator">-</span>report<span class="token punctuation">.</span>html<span class="token punctuation">.</span>twig</code> (from both system and stable9 theme), and <code class=" language-php"><span class="token function">template_preprocess_authorize_report</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>
</li></ol>
<p>There is no replacement. <a href="https://www.drupal.org/docs/develop/using-composer" rel="nofollow">Use composer to manage the code for Drupal sites</a>.</p>
<p><code class=" language-php"><span class="token function">update_clear_update_disk_cache</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> used to be called via <code class=" language-php"><span class="token function">hook_cron</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>, but that is no longer the case. Sites that haven't used the Update Manager should not have any files in this cache. A post update method is provided, <code class=" language-php"><span class="token function">update_post_update_clear_disk_cache</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>, to remove directories named "<code class=" language-php">update<span class="token operator">-</span>cache<span class="token operator">-</span><span class="token operator">*</span></code>" and "<code class=" language-php">update<span class="token operator">-</span>extraction<span class="token operator">-</span><span class="token operator">*</span></code>" (and all of their contents) in the site's temporary directory on disk.</p>

---

## [NodeForm and NodeTypeForm have moved to the Drupal\node\Form namespace](https://www.drupal.org/node/3517871)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The forms, <code class=" language-php">Drupal\<span class="token package">node<span class="token punctuation">\</span>NodeForm</span></code> and <code class=" language-php">Drupal\<span class="token package">node<span class="token punctuation">\</span>NodeTypeForm</span></code> have moved to the <code class=" language-php">node<span class="token operator">/</span>src<span class="token operator">/</span>Form</code> directory. Previously, they were in the root of node module, <code class=" language-php">node<span class="token operator">/</span>src</code>. They are moved for consistency.</p>
<p>Their namespaces have changed:<br>
<code class=" language-php">Drupal\<span class="token package">node<span class="token punctuation">\</span>NodeForm</span></code> is now <code class=" language-php">Drupal\<span class="token package">node<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>NodeForm</span></code><br>
<code class=" language-php">Drupal\<span class="token package">node<span class="token punctuation">\</span>NodeTypeForm</span></code> is now <code class=" language-php">Drupal\<span class="token package">node<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>NodeTypeForm</span></code></p>

---

## [Added component variants to SDC](https://www.drupal.org/node/3517062)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>In design systems, a <em>variant</em> allow grouping multiple component properties into a predefined set. The <em>variant</em> can then be used as a shortcut to render a component.</p>
<p>In SDC, front-end developers could previously define a <em>variant</em> as a <em>prop</em>, but this approach did not support custom titles or descriptions to convey the variant’s purpose.</p>
<p>This change allows component authors to specify a <code class=" language-php">title</code> and <code class=" language-php">description</code> that differ from the machine name, enabling more meaningful labels and detailed explanations of each variant’s purpose.</p>
<p>Now, you can use <code class=" language-php">variants</code> as a property at the root of your component declaration:</p>
<pre class="codeblock language-php"><code class=" language-php">name<span class="token punctuation">:</span>  Card
variants<span class="token punctuation">:</span>
  primary<span class="token punctuation">:</span>
    title<span class="token punctuation">:</span> Primary
    description<span class="token punctuation">:</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
  secondary<span class="token punctuation">:</span>
    title<span class="token punctuation">:</span> Secondary
    description<span class="token punctuation">:</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
props<span class="token punctuation">:</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
slots<span class="token punctuation">:</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
</code></pre><p>
In your component twig template, you can use this as a regular <em>prop</em>:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token punctuation">{</span><span class="token operator">%</span> <span class="token keyword keyword-if">if</span> variant is not empty <span class="token operator">%</span><span class="token punctuation">}</span>
  <span class="token punctuation">{</span><span class="token operator">%</span> set attributes <span class="token operator">=</span> attributes<span class="token punctuation">.</span><span class="token function">addClass</span><span class="token punctuation">(</span><span class="token string">'card-'</span> <span class="token operator">~</span> variant<span class="token punctuation">)</span> <span class="token operator">%</span><span class="token punctuation">}</span>
<span class="token punctuation">{</span><span class="token operator">%</span> <span class="token keyword keyword-endif">endif</span> <span class="token operator">%</span><span class="token punctuation">}</span>
</code></pre><p>
When rendering a component from another twig template, you can specify the <em>variant</em> with:</p>
<pre class="codeblock language-php"><code class=" language-php">      <span class="token punctuation">{</span><span class="token operator">%</span> <span class="token keyword keyword-include">include</span> <span class="token string">"mytheme:card"</span> with <span class="token punctuation">{</span>
        variant<span class="token punctuation">:</span> <span class="token string">'primary'</span><span class="token punctuation">,</span>
        attributes<span class="token punctuation">:</span> <span class="token function">create_attribute</span><span class="token punctuation">(</span><span class="token punctuation">{</span><span class="token string">'class'</span><span class="token punctuation">:</span> <span class="token punctuation">[</span><span class="token string">'banner__title'</span><span class="token punctuation">]</span><span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
        label<span class="token punctuation">:</span> content<span class="token punctuation">.</span>field_title<span class="token punctuation">,</span>
      <span class="token punctuation">}</span> only <span class="token operator">%</span><span class="token punctuation">}</span>
</code></pre><p>
Using the render arrays API (e.g. from a controller), you can use:</p>
<pre class="codeblock language-php"><code class=" language-php">      <span class="token variable">$build</span><span class="token punctuation">[</span><span class="token string">'content'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
        <span class="token string">'#type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'component'</span><span class="token punctuation">,</span>
        <span class="token string">'#component'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'mytheme:card'</span><span class="token punctuation">,</span>
        <span class="token string">'#variant'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'secondary'</span><span class="token punctuation">,</span>
        <span class="token string">'#props'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
          <span class="token string">'label'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'My custom card'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
        <span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre><p>
Note that the concept of variants introduced in this change is not related to the broader use of "variants" in design systems and tools like Storybook, where the term typically refers to a combination of prop values and is commonly used for demonstrating components.</p>

---

## [Not providing an attribute class for a plugin that that uses annotion based discovery is now deprecated](https://www.drupal.org/node/3522776)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Plugin types that use annotations for discovery can provide an attribute class since Drupal 10.2.</p>
<p>Not providing an attribute class is now deprecated, and an attribute class will be required in Drupal 12.0. It is and will still be possible to also provide an annotation class for BC until Drupal 13, only then will support for annotations be removed completely.</p>
<p>See <a href="https://www.drupal.org/node/3395582" rel="nofollow">Plugin types should use PHP attributes instead of annotations</a> and <a href="https://www.drupal.org/node/3395575" rel="nofollow">Plugin implementations should use PHP attributes instead of annotations</a> for more information.</p>

---

## [The $root parameter for \Drupal\Core\Database\Connection::createConnectionOptionsFromUrl is deprecated](https://www.drupal.org/node/3511287)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The $root parameter for \Drupal\Core\Database\Connection::createConnectionOptionsFromUrl() and \Drupal\Core\Database\Database::convertDbUrlToConnectionInfo() is deprecated, it is no longer needed by any implementation.</p>
<p>As the argument was required, pass NULL in any calls; this can then be removed in Drupal 12.</p>

---

## [BlockSettings migration plugin now requires the block plugin manager](https://www.drupal.org/node/3522023)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php">block_settings</code> migration plugin (used for migrations from Drupal 6 and Drupal 7, implemented by <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>block<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>process<span class="token punctuation">\</span>BlockSettings</span></code>) now requires the block plugin manager service (<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Block<span class="token punctuation">\</span>BlockManagerInterface</span></code>) to be passed to its constructor so that it can normalize block settings after processing them.</p>

---

## [hooks_converted parameter and StopProceduralHookScan attributes have been renamed.](https://www.drupal.org/node/3498595)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>To ensure that all preprocess functions are picked up, the <code class=" language-php">hooks_converted</code> parameter has been renamed to <code class=" language-php">skip_procedural_hook_scan</code> and the <code class=" language-php">StopProceduralHookScan</code> attribute has been renamed to <code class=" language-php">ProceduralHookScanStop</code>.</p>
<p>When updating modules to 11.2, if the module has used the old parameter or attribute, it should be sufficient to do a find and replace with the new name after ensuring that all preprocess functions are converted to OOP. A module can also chose to add both parameters.</p>
<p>Note: The services parameter and especially the attribute are mostly useful for large modules with many functions (e.g. legacy hooks to support 11.0 and earlier versions). For smaller modules, the performance improvement are very likely tiny and probably not measurable at all. Both are entirely optional and purely a performance optimization.</p>
<p><strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php">parameters<span class="token punctuation">:</span>
  <span class="token constant">MODULE_NAME</span><span class="token punctuation">.</span>hooks_converted<span class="token punctuation">:</span> <span class="token boolean">true</span>
</code></pre><p>
<strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php">parameters<span class="token punctuation">:</span>
  <span class="token constant">MODULE_NAME</span><span class="token punctuation">.</span>skip_procedural_hook_scan<span class="token punctuation">:</span> <span class="token boolean">true</span>
</code></pre><p>
<strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>StopProceduralHookScan</span><span class="token punctuation">;</span>

<span class="token shell-comment comment">#[StopProceduralHookScan]</span>
<span class="token keyword keyword-function">function</span> <span class="token function">locale_translation_batch_version_check</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
</code></pre><p>
<strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>ProceduralHookScanStop</span><span class="token punctuation">;</span>

<span class="token shell-comment comment">#[ProceduralHookScanStop]</span>
<span class="token keyword keyword-function">function</span> <span class="token function">locale_translation_batch_version_check</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
</code></pre>

---

## [DefaultPluginManager uses attribute before annotation during discovery](https://www.drupal.org/node/3521588)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Attributes are parsed first, so for plugin classes that have both, the definition will be parsed from attributes.</p>
<p><a href="https://www.drupal.org/project/drupal/issues/3252386" rel="nofollow">When plugin discovery by PHP attributes was introduced</a>, annotations in the plugin classes were parsed before attributes. This meant that if a plugin class had both the plugin attribute and plugin annotation (for backwards compatibility), then the plugin definition would be parsed from the annotation values, not the attribute values.</p>

---

## [Hook implementations can be ordered with an order parameter](https://www.drupal.org/node/3493962)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php"><span class="token shell-comment comment">#[Hook]</span></code> attribute has an order argument to allow hook implementations to be ordered.</p>
<p>The order parameter accepts the following simple ordering options.<br>
<code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Order<span class="token punctuation">::</span></span>First</code><br>
<code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Order<span class="token punctuation">::</span></span>Last</code></p>
<p>The order parameter also accepts the following complex ordering options which also accept parameters.<br>
<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>OrderBefore</span></code><br>
<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>OrderAfter</span></code></p>
<p><code class=" language-php"><span class="token function">OrderBefore</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and <code class=" language-php"><span class="token function">OrderAfter</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> accept the following parameters:<br>
<code class=" language-php">modules<span class="token punctuation">:</span></code> an array of modules to order before or after.<br>
<code class=" language-php">classesAndMethods<span class="token punctuation">:</span></code> an array of arrays of classes and methods to order before or after.</p>
<p>See multiple ordering operations below for information on how multiple modules implementing Order::First is resolved.</p>
<h2>Examples</h2>
<h3>Order the hook first</h3>
<p><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'some_hook'</span><span class="token punctuation">,</span> order<span class="token punctuation">:</span> <span class="token scope">Order<span class="token punctuation">::</span></span>First<span class="token punctuation">)</span><span class="token punctuation">]</span></code></p>
<h3>Order the hook last</h3>
<p><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'some_hook'</span><span class="token punctuation">,</span> order<span class="token punctuation">:</span> <span class="token scope">Order<span class="token punctuation">::</span></span>Last<span class="token punctuation">)</span><span class="token punctuation">]</span></code></p>
<h3>Order the hook before another module's implementation</h3>
<p><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'some_hook'</span><span class="token punctuation">,</span> order<span class="token punctuation">:</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">OrderBefore</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'other_module'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">]</span></code></p>
<h3>Order the hook after another module's implementation</h3>
<p><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'some_hook'</span><span class="token punctuation">,</span> order<span class="token punctuation">:</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">OrderAfter</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'other_module'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">]</span></code></p>
<h3>Order the hook before another module's implementation by specifying class and method</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'some_hook'</span><span class="token punctuation">,</span> 
  order<span class="token punctuation">:</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">OrderBefore</span><span class="token punctuation">(</span>
    classesAndMethods<span class="token punctuation">:</span> <span class="token punctuation">[</span>
      <span class="token punctuation">[</span><span class="token scope">Foo<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span> <span class="token string">'someMethod'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token punctuation">[</span><span class="token scope">Bar<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span> <span class="token string">'someOtherMethod'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span>
  <span class="token punctuation">)</span>
<span class="token punctuation">)</span><span class="token punctuation">]</span>
</code></pre><h3>Order the hook after another module's implementation by specifying class and method</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'some_hook'</span><span class="token punctuation">,</span> 
  order<span class="token punctuation">:</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">OrderAfter</span><span class="token punctuation">(</span>
    classesAndMethods<span class="token punctuation">:</span> <span class="token punctuation">[</span>
      <span class="token punctuation">[</span><span class="token scope">Foo<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span> <span class="token string">'someMethod'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token punctuation">[</span><span class="token scope">Bar<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span> <span class="token string">'someOtherMethod'</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span>
  <span class="token punctuation">)</span>
<span class="token punctuation">)</span><span class="token punctuation">]</span>
</code></pre><h2>Multiple ordering operations</h2>
<p>By default hooks are ordered in module order which is defined by module weight, then alphabetically.<br>
Ordering operations are applied by module order as well. <a href="https://www.drupal.org/node/3497308" rel="nofollow">Reorder operations</a> happen after all Order operations have been applied and they occur in module order as well.<br>
Order operations are applied sequentially and apply to the current order. If multiple modules say they want their hook to run first, the one that is last in module order will be first.</p>
<p>For example if we have modules A, B, and C that implement some_hook.<br>
Default order:</p>
<ul>
<li>A::some_hook</li>
<li>B::some_hook</li>
<li>C::some_hook</li>
</ul>
<p>If B and C both apply Order::First then the final order will be:</p>
<ul>
<li>C::some_hook</li>
<li>B::some_hook</li>
<li>A::some_hook</li>
</ul>
<p>We start with A, then B, then C. When B's order operation is applied B says it wants to be first so we get B, A, C. Then C's order operation is applied and C says it wants to be first so we end up with C, B, A.</p>
<h2>hook_module_implements_alter</h2>
<p>If you ordered your hooks using <code class=" language-php">hook_module_implements_alter</code> see <a href="https://www.drupal.org/node/3496788" rel="nofollow">hook_module_implements_alter requires the #[LegacyModuleImplementsAlter] attribute and ordering must be done using new functionality</a> for more information.</p>

---

## [Hook implementations can now be removed with a #[RemoveHook] attribute.](https://www.drupal.org/node/3496786)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The new <code class=" language-php"><span class="token shell-comment comment">#[RemoveHook]</span></code> attribute can be used when you want to remove the implementation of a hook in another module. It accepts the following required parameters:</p>
<ul>
<li>hook</li>
<li>class</li>
<li>method</li>
</ul>
<p>You may place the <code class=" language-php"><span class="token shell-comment comment">#[RemoveHook]</span></code> attribute on any class or method in the Hook namespace, however for organizational purposes it is recommended you put it on the method that requires you to remove the other implementation.</p>
<p>Example conversion for <code class=" language-php">navigation_module_implementation_alter</code></p>
<p><strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-function">function</span> <span class="token function">navigation_module_implements_alter</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$implementations</span><span class="token punctuation">,</span> <span class="token variable">$hook</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$hook</span> <span class="token operator">==</span> <span class="token string">'help'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment" spellcheck="true">// We take over the layout_builder hook_help().</span>
    <span class="token function">unset</span><span class="token punctuation">(</span><span class="token variable">$implementations</span><span class="token punctuation">[</span><span class="token string">'layout_builder'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre><p>
<strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package"><span class="token punctuation">\</span>Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>RemoveHook</span><span class="token punctuation">;</span>
<span class="token shell-comment comment">#[RemoveHook(</span><span class="token string">'help'</span><span class="token punctuation">,</span> <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> <span class="token scope">LayoutBuilderHooks<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span> method<span class="token punctuation">:</span> <span class="token string">'help'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
</code></pre><p>
For converting other functionality of <code class=" language-php">hook_module_implements_alter</code> see <a href="https://www.drupal.org/node/3496788" rel="nofollow">hook_module_implements_alter requires the #[LegacyHook] attribute and ordering must be done using new functionality</a>.</p>

---

## [hook_module_implements_alter requires the #[LegacyModuleImplementsAlter] attribute](https://www.drupal.org/node/3496788)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The functionality of <code class=" language-php">hook_module_implements_alter</code> is replaced with two new attributes and an <code class=" language-php">order</code> parameter on the <code class=" language-php"><span class="token shell-comment comment">#[Hook]</span></code> attribute. Ordering <em>must</em> be done using new functionality.</p>
<h4>
Update existing <code class=" language-php">hook_module_implements_alter</code> code</h4>
<p>This is not a typical deprecation and requires care for your module to support versions of Drupal older than 11.2</p>
<p>Follow these steps to update your module in a backwards compatible way;</p>
<ol>
<li>Achieve the same functionality as your <code class=" language-php">hook_module_implements_alter</code>. Dynamic ordering is currently not supported.
</li><li> Add the <code class=" language-php"><span class="token shell-comment comment">#[LegacyModuleImplementsAlter]</span></code> to your implementation of <code class=" language-php">hook_module_implements_alter</code>. Once the minimum supported version of Drupal for the module is 11.2 or larger you may delete the entire function and attribute.
</li></ol>
<p>These change records provide examples for updating <code class=" language-php">hook_module_implements_alter</code> implementations.</p>
<ul>
<li><a href="https://www.drupal.org/node/3493962" rel="nofollow">Order hook implementations  with the order parameter.</a></li>
<li><a href="https://www.drupal.org/node/3497308" rel="nofollow">Reorder hook implementations in other modules with the #[ReOrderHook] attribute.</a></li>
<li><a href="https://www.drupal.org/node/3496786" rel="nofollow">Remove hook implementations the #[RemoveHook] attribute.</a></li>
</ul>

---

## [Reorder hook implementations in other modules with the #[ReOrderHook] attribute](https://www.drupal.org/node/3497308)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>In the rare case when a module wants to change when the implementation of a hook by another module executes, the <code class=" language-php"><span class="token shell-comment comment">#[ReOrderHook]</span></code> attribute can be used. This attribute accepts the following required parameters:</p>
<ul>
<li>hook</li>
<li>class</li>
<li>method</li>
<li>order</li>
</ul>
<p>See <a href="https://www.drupal.org/node/3493962" rel="nofollow">Hook implementations can now be ordered with an order parameter</a> for the possible value of <code class=" language-php">order</code></p>
<p>You may place the <code class=" language-php"><span class="token shell-comment comment">#[ReOrderHook]</span></code> attribute on any class or method in the Hook namespace. However we recommend you put it on the implementation that requires you to reorder the other implementation.</p>
<h4>Example</h4>
<p>From the Workspaces module hook implemention, <code class=" language-php">workspaces_module_implementation_alter</code>, move the implementation of the Content Moderation's <code class=" language-php">entity_presave</code> hook to before the Workspaces module implementation.</p>
<p><strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-function">function</span> <span class="token function">workspaces_module_implements_alter</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$implementations</span><span class="token punctuation">,</span> <span class="token variable">$hook</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$hook</span> <span class="token operator">===</span> <span class="token string">'entity_presave'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment" spellcheck="true">// Move Content Moderation's implementation before Workspaces, so we can</span>
    <span class="token comment" spellcheck="true">// alter the publishing status for the default revision.</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token function">isset</span><span class="token punctuation">(</span><span class="token variable">$implementations</span><span class="token punctuation">[</span><span class="token string">'content_moderation'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$implementation</span> <span class="token operator">=</span> <span class="token variable">$implementations</span><span class="token punctuation">[</span><span class="token string">'content_moderation'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
      <span class="token variable">$implementations</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token string">'content_moderation'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$implementation</span><span class="token punctuation">]</span> <span class="token operator">+</span> <span class="token variable">$implementations</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span></code></pre><p>
<strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package"><span class="token punctuation">\</span>Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>ReOrderHook</span><span class="token punctuation">;</span>
<span class="token shell-comment comment">#[ReOrderHook(</span><span class="token string">'entity_presave'</span><span class="token punctuation">,</span>
  <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> <span class="token scope">ContentModerationHooks<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span>
  method<span class="token punctuation">:</span> <span class="token string">'entityPresave'</span><span class="token punctuation">,</span>
  order<span class="token punctuation">:</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">OrderBefore</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'workspaces'</span><span class="token punctuation">]</span><span class="token punctuation">)</span>
<span class="token punctuation">)</span><span class="token punctuation">]</span>
</code></pre><p>
For converting other functionality of <code class=" language-php">hook_module_implements_alter</code> see <a href="https://www.drupal.org/node/3496788" rel="nofollow">hook_module_implements_alter requires the #[LegacyHook] attribute and ordering must be done using new functionality</a>.</p>

---

## [Procedural hooks are ordered before object oriented hooks for a given module](https://www.drupal.org/node/3515207)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<h3>Background</h3>
<p>The order of hook implementations for a hook follows the order of the modules that provide these implementations.<br>
This module order is based on module weights and alphabetical order of module machine names, and it can be altered with hook_module_implements_alter().</p>
<p>Since <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3485896" title="Status: Closed (fixed)">#3485896: Hook ordering across OOP, procedural and with extra types i.e replace hook_module_implements_alter</a></span>, the order of individual implementations can be changed with newly introduced attributes, or with a new parameter in the #[Hook] attribute. (see other change records).</p>
<p><strong>Before</strong><br>
When one module provides multiple implementations for the same hook, their order relative to each other is not guaranteed. It depends on the order of discovery, which can be system-dependent.</p>
<p>Especially, when one module provides one procedural hook implementation and one or more OOP hook implementations for the <em>same hook</em>, it is not guaranteed whether the procedural implementation will run first, last, or in between the OOP implementations.</p>
<p><strong>After</strong><br>
When invoking a hook, any procedural hook implementation will be executed before any OOP hook implementations <em>of the same module</em>, unless overridden by order settings in attributes.</p>
<h3>Example</h3>
<p>Prior to this change, hook implementations of hook_myhook() might be ordered like this:</p>
<ul>
<li>module_a_myhook()
</li>
<li>Drupal\module_a\Hook\Hooks::myHook()
</li>
<li>Drupal\module_b\Hook\Hooks::myHook()
</li>
<li>module_b_myhook()
</li>
</ul>
<p>After this change, they would be ordered like this:</p>
<ul>
<li>module_a_myhook()
</li>
<li>Drupal\module_a\Hook\Hooks::myHook()
</li>
<li>module_b_myhook()
</li>
<li>Drupal\module_b\Hook\Hooks::myHook()
</li>
</ul>

---

## [The update.module has been renamed back to 'Update Status'](https://www.drupal.org/node/3521054)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The update.module included in Drupal Core used to be called the "Update Manager". However, the "manager" aspects of it did not work with composer managed sites, which is now a requirement. After all of these issues, the update.module no longer "manages" any updates:</p>
<ul>
<li><span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3417136" title="Status: Closed (fixed)">#3417136: Remove adding an extension via a URL</a></span></li>
<li><span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3491731" title="Status: Closed (fixed)">#3491731: [META] Remove the ability to update modules and themes via authorize.php</a></span>
<ul>
<li><span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3502973" title="Status: Closed (fixed)">#3502973: Remove UI and routes for the ability to update modules and themes via update.module and authorize.php</a></span></li>
<li><span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3502974" title="Status: Closed (fixed)">#3502974: Deprecate authorize.php, the FileTransfer and Updater systems</a></span></li>
</ul>
</li>
</ul>
<p>It is informational only, providing the <strong>Available Updates</strong> status report. Therefore, the human readable name has been changed back to "Update Status", to make way for a new and improved "Update Manager" via <span class="project-issue-issue-link project-issue-status-info project-issue-status-13"><a href="https://www.drupal.org/project/drupal/issues/3253158" title="Status: Needs work">#3253158: Add Alpha level Experimental Update Manager module</a></span>.</p>

---

## [Various classes and methods renamed from 'Stage' to 'Sandbox' in package manager](https://www.drupal.org/node/3521441)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A number of classes and methods have been renamed in the Package Manager module in order to remove confusion about their purpose and intended use.</p>
<pre class="codeblock language-php"><code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Event<span class="token operator">/</span>RequireEventTrait<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> EventWithPackageListTrait<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Event<span class="token operator">/</span>StageEvent<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxEvent<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Event<span class="token operator">/</span>PreOperationStageEvent<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxValidationEvent<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Exception<span class="token operator">/</span>StageFailureMarkerException<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> FailureMarkerExistsException<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Exception<span class="token operator">/</span>StageEventException<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxEventException<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Exception<span class="token operator">/</span>StageException<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxException<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Exception<span class="token operator">/</span>StageOwnershipException<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxOwnershipException<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>StageBase<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxManagerBase<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Validator<span class="token operator">/</span>StagedDBUpdateValidator<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxDatabaseUpdatesValidator<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>src<span class="token operator">/</span>Validator<span class="token operator">/</span>StageNotInActiveValidator<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxDirectoryValidator<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>tests<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager_test_validation<span class="token operator">/</span>src<span class="token operator">/</span>StagedDatabaseUpdateValidator<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> TestSandboxDatabaseUpdatesValidator<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>tests<span class="token operator">/</span>src<span class="token operator">/</span>Kernel<span class="token operator">/</span>StageBaseTest<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxManagerBaseTest<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>tests<span class="token operator">/</span>src<span class="token operator">/</span>Unit<span class="token operator">/</span>RequireEventTraitTest<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> EventWithPackageListTraitTest<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>tests<span class="token operator">/</span>src<span class="token operator">/</span>Unit<span class="token operator">/</span>StageNotInActiveValidatorTest<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxDirectoryValidatorTest<span class="token punctuation">.</span>php
core<span class="token operator">/</span>modules<span class="token operator">/</span>package_manager<span class="token operator">/</span>tests<span class="token operator">/</span>src<span class="token operator">/</span>Unit<span class="token operator">/</span>StageBaseTest<span class="token punctuation">.</span>php <span class="token operator">=</span><span class="token operator">&gt;</span> SandboxManagerBaseTest<span class="token punctuation">.</span>php

</code></pre>

---

## [New config.schema_checker service to check configuration schema in a development setting](https://www.drupal.org/node/2828126)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>This change adds <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>DevelopmentLenientConfigSchemaChecker</span></code> and the <code class=" language-php"> config<span class="token punctuation">.</span>schema_checker</code> service to validate config schema on save. Only enabled in development.services.yml by default to help identify issues while developing.</p>

---

## [\Drupal\migrate\Attribute\MigrateSource does not include the source_module property](https://www.drupal.org/node/3306373)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Only Migrate source plugins that extend <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>DrupalSqlBase</span></code> are required to declare <code class=" language-php">source_module</code> in their annotation. Previously, the requirement extended to all Migrate source plugins, including those not being used to migrate a Drupal 6 or Drupal 7 source site.</p>
<p>Source plugins can use attributes (<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>MigrateSource</span></code>) or annotations (<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Annotation<span class="token punctuation">\</span>MigrateSource</span></code>). The attribute class does not include the <code class=" language-php">source_module</code>, so only annotation-based source plugins can declare that property in their definitions. Any source plugin supports <code class=" language-php">source_module</code> in its configuration.</p>
<p>If you use <code class=" language-php">source_module</code> in Migrate source plugins that do not extend <code class=" language-php">DrupalSqlBase</code>, then the property <code class=" language-php">source_module</code> can still be added and used according to your needs. However, the plugin manager in the <code class=" language-php">migrate_drupal</code> module may issue a deprecation message if you specify <code class=" language-php">source_module</code> when it is not needed. You can avoid the deprecation message by including either "Drupal 6" or "Drupal 7" in <code class=" language-php">migration_tags</code> in the migration that uses the source plugin. You can also customize the list of relevant tags in the <code class=" language-php">migrate_drupal<span class="token punctuation">.</span>settings</code> config object.</p>
<h3>Before Drupal 11.2.0</h3>
<p>All source plugins use annotations. The <code class=" language-php">source_module</code> property is supported, and it is required for source plugins that extend <code class=" language-php">DrupalSqlBase</code>.</p>
<p>For example, the <code class=" language-php">embedded_data</code> source plugin is defined in <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>migrate<span class="token operator">/</span>src<span class="token operator">/</span>Plugin<span class="token operator">/</span>migrate<span class="token operator">/</span>source<span class="token operator">/</span>EmbeddedDataSource<span class="token punctuation">.</span>php</code>:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>MigrationInterface</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * Allows source data to be defined in the configuration of the source plugin.
 * ...
 *
 * @MigrateSource(
 *   id = "embedded_data",
 *   source_module = "migrate"
 * )
 */</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">EmbeddedDataSource</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">SourcePluginBase</span> <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// ...</span>
<span class="token punctuation">}</span>
</code></pre><p>
The <code class=" language-php">block_content_body_field</code> migration, defined in <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>block_content<span class="token operator">/</span>migrations<span class="token operator">/</span>block_content_body_field<span class="token punctuation">.</span>yml</code>, uses <code class=" language-php">embedded_data</code> and overrides <code class=" language-php">source_module</code> in its configuration:</p>
<pre class="codeblock language-php"><code class=" language-php">id<span class="token punctuation">:</span> block_content_body_field
label<span class="token punctuation">:</span> Block content body field configuration
migration_tags<span class="token punctuation">:</span>
  <span class="token operator">-</span> Drupal <span class="token number">6</span>
  <span class="token operator">-</span> Drupal <span class="token number">7</span>
  <span class="token operator">-</span> Configuration
source<span class="token punctuation">:</span>
  plugin<span class="token punctuation">:</span> embedded_data
  <span class="token shell-comment comment"># ...</span>
  source_module<span class="token punctuation">:</span> block
<span class="token shell-comment comment"># ...</span>
</code></pre><p>
Similarly, the <code class=" language-php">variable</code> plugin, defined in <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>migrate_drupal<span class="token operator">/</span>src<span class="token operator">/</span>Plugin<span class="token operator">/</span>migrate<span class="token operator">/</span>source<span class="token operator">/</span>Variable<span class="token punctuation">.</span>php</code>:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityTypeManagerInterface</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>State<span class="token punctuation">\</span>StateInterface</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>MigrationInterface</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * Drupal 6/7 variable source from database.
 * ...
 * @MigrateSource(
 *   id = "variable",
 *   source_module = "system",
 * )
 */</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">Variable</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">DrupalSqlBase</span> <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// ...</span>
<span class="token punctuation">}</span>
</code></pre><p>
The <code class=" language-php">d7_node_settings</code> migration, defined in <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>node<span class="token operator">/</span>migrations<span class="token operator">/</span>d7_node_settings<span class="token punctuation">.</span>yml</code>, uses the <code class=" language-php">variable</code> source plugin and overrides <code class=" language-php">source_module</code>:</p>
<pre class="codeblock language-php"><code class=" language-php">id<span class="token punctuation">:</span> d7_node_settings
label<span class="token punctuation">:</span> Node configuration
migration_tags<span class="token punctuation">:</span>
  <span class="token operator">-</span> Drupal <span class="token number">7</span>
  <span class="token operator">-</span> Configuration
source<span class="token punctuation">:</span>
  plugin<span class="token punctuation">:</span> variable
  variables<span class="token punctuation">:</span>
    <span class="token operator">-</span> node_admin_theme
  source_module<span class="token punctuation">:</span> node
<span class="token shell-comment comment">#...</span>
</code></pre><h3>Starting in Drupal 11.2.0</h3>
<p>The <code class=" language-php">embedded_data</code> source plugin uses attributes and does not specify <code class=" language-php">source_module</code>:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>MigrateSource</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>MigrationInterface</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * Allows source data to be defined in the configuration of the source plugin.
 * ...
 */</span>
<span class="token shell-comment comment">#[MigrateSource(</span><span class="token string">'embedded_data'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">EmbeddedDataSource</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">SourcePluginBase</span> <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// ...</span>
<span class="token punctuation">}</span>
</code></pre><p>
There is no change in the <code class=" language-php">block_content_body_field</code> migration. It still uses the <code class=" language-php">embedded_data</code> source plugin, and it still declares <code class=" language-php">source_module</code> in its configuration.</p>
<p>Since the <code class=" language-php">variable</code> source plugin extends <code class=" language-php">DrupalSqlBase</code>, there is no change. It still uses annotations, and it still specifies <code class=" language-php">source_module</code> in its definition.</p>
<p>There is also no change in the <code class=" language-php">d7_node_settings</code> migration. It still uses the <code class=" language-php">variable</code> source plugin, and it still overrides <code class=" language-php">source_module</code> in its configuration.</p>

---

## [New AtLeastOneOf constraint for config validation](https://www.drupal.org/node/3517913)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>It is now possible to use a <code class=" language-php">AtLeastOneOf</code> constraints. This allows for validation of multiple constraints of which one should validate. </p>
<p>This allows for constraints like:</p>
<pre class="codeblock language-php"><code class=" language-php">            constraints<span class="token punctuation">:</span>
              AtLeastOneOf<span class="token punctuation">:</span>
                constraints<span class="token punctuation">:</span>
                  <span class="token operator">-</span> PluginExists<span class="token punctuation">:</span>
                      manager<span class="token punctuation">:</span> plugin<span class="token punctuation">.</span>manager<span class="token punctuation">.</span>menu<span class="token punctuation">.</span>link
                      <span class="token keyword keyword-interface">interface</span><span class="token punctuation">:</span> <span class="token string">'Drupal\Core\Menu\MenuLinkInterface'</span>
                  <span class="token operator">-</span> IdenticalTo<span class="token punctuation">:</span>
                      value<span class="token punctuation">:</span> <span class="token string">''</span>
</code></pre><p>
Which then does validation on both, but only fails if no validation succeeds.</p>

---

## [New filters 'Is empty (NULL)' and 'Is not empty (NOT NULL)' added to Views field filter operators](https://www.drupal.org/node/3370509)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p><code class=" language-php"><span class="token string">"Is empty (NULL)"</span></code> and <code class=" language-php"><span class="token string">"Is not empty (NOT NULL)"</span></code> options available on Views UI filters for all Not Required fields.</p>
<p><strong>Before</strong><br>
<img src="/files/issues/2021-07-16/3098560_before_patch_pic.png" alt="Before MR"></p>
<p><strong>After</strong><br>
<img src="/files/issues/2021-07-16/3098560_after_patch_pic.png" alt="After MR"></p>

---

## [New ClassResolverConstraint to validate bases on service or instantiated class](https://www.drupal.org/node/3519434)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The new ClassResolverConstraint checks if a method on a service or instantiated object returns true.</p>
<p>For example to call the method 'isValidScheme' on the service  'stream_wrapper_manager', use: ['stream_wrapper_manager', 'isValidScheme'].</p>
<p>It is also possible to use a class if it implements ContainerInjectionInterface. It will use the ClassResolver to resolve the class and return an instance. Then it will call the configured method on that instance.</p>
<p>The called method should return TRUE when the result is valid. All other values will be considered as invalid.</p>

---

## [ExtensionMimeTypeGuesser extension and mimetype maps are split off into a new MimeTypeMapInterface and service](https://www.drupal.org/node/3494040)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core

### Description

<p>Previously a <code class=" language-php"><span class="token variable">$defaultMapping</span></code> protected property existed on <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>File<span class="token punctuation">\</span>MimeType<span class="token punctuation">\</span>ExtensionMimeTypeGuesser</span></code>. However, services that just needed to change the map of mimetype to extensions (and visa versa) had to implement a whole alternative guesser.</p>
<p>The following methods on <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>File<span class="token punctuation">\</span>MimeType<span class="token punctuation">\</span>ExtensionMimeTypeGuesser</span> </code> are now deprecated:</p>
<ul>
<li><code class=" language-php"><span class="token function">setMapping</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">getMapping</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
</ul>
<p>The map is now provided by a separate <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>File<span class="token punctuation">\</span>MimeType<span class="token punctuation">\</span>MimeTypeMapInterface</span></code> property, with a <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>File<span class="token punctuation">\</span>MimeType<span class="token punctuation">\</span>DefaultMimeTypeMap</span></code> implementation and service. This allows different implementations to override the map service rather than the whole guesser service.</p>
<p>The protected <code class=" language-php"><span class="token variable">$defaultMapping</span></code> property on <code class=" language-php">ExtensionMimeTypeGuesser</code> has been removed. Users wanting to replace the mapping should inject their own implementation of <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>File<span class="token punctuation">\</span>MimeType<span class="token punctuation">\</span>MimeTypeMapInterface</span></code> instead.</p>
<p>The <code class=" language-php"><span class="token function">hook_file_mimetype_mapping_alter</span><span class="token punctuation">(</span></code>) hook is also deprecated and will be removed in Drupal 12. Users should implement an <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>File<span class="token punctuation">\</span>Event<span class="token punctuation">\</span>MimeTypeMapLoadedEvent</span></code> listener to alter the map instead.</p>

---

## [The PostgreSQL override of entityQuery is now in the pgsql module](https://www.drupal.org/node/3488580)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The PostgreSQL override of the entityQuery is moved to the pgsql module. The PostgreSQL override of the entityQuery now lives in the namespace <code class=" language-php">Drupal\<span class="token package">pgsql<span class="token punctuation">\</span>EntityQuery</span></code>.</p>
<p>The 2 classes <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Query<span class="token punctuation">\</span>Sql<span class="token punctuation">\</span>pgsql<span class="token punctuation">\</span>QueryFactory</span></code> and <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Query<span class="token punctuation">\</span>Sql<span class="token punctuation">\</span>pgsql<span class="token punctuation">\</span>Condition</span></code> are deprecated and will be removed before Drupal 12.0.0. </p>
<p>This is continuation of moving database drivers, and the specific code, to their own module.</p>

---

## [The d8_config migrate source plugin is replaced by the config_entity plugin](https://www.drupal.org/node/3508578)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The class <code class=" language-php">Drupal\<span class="token package">migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>d8<span class="token punctuation">\</span>Config</span></code> is deprecated. This class provides the <code class=" language-php">d8_config</code> migrate source plugin.</p>
<p>The new class <code class=" language-php">Drupal\<span class="token package">migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>ConfigEntity</span></code> provides the <code class=" language-php">config_entity</code> migrate source plugin, which is essentially identical to the <code class=" language-php">d8_config</code> plugin.</p>
<h3>Before Drupal 11.2.0</h3>
<p>A migration can declare the <code class=" language-php">d8_config</code> source plugin:</p>
<pre class="codeblock language-php"><code class=" language-php">source
  plugin<span class="token punctuation">:</span> d8_config
  names<span class="token punctuation">:</span>
    <span class="token operator">-</span> node<span class="token punctuation">.</span>type<span class="token punctuation">.</span>article
    <span class="token operator">-</span> node<span class="token punctuation">.</span>type<span class="token punctuation">.</span>page
</code></pre><p>
The <code class=" language-php">migrate_drupal</code> module must be enabled in order to use this migration.</p>
<h3>Drupal 11.2.0 and later</h3>
<p>A migration can declare the <code class=" language-php">config_entity</code> source plugin:</p>
<pre class="codeblock language-php"><code class=" language-php">source
  plugin<span class="token punctuation">:</span> config_entity
  names<span class="token punctuation">:</span>
    <span class="token operator">-</span> node<span class="token punctuation">.</span>type<span class="token punctuation">.</span>article
    <span class="token operator">-</span> node<span class="token punctuation">.</span>type<span class="token punctuation">.</span>page
</code></pre><p>
The <code class=" language-php">migrate</code> module must be enabled in order to use this migration.</p>
<h3>Contrib and Custom Source Plugins</h3>
<p>The deprecated class extends <code class=" language-php">Drupal\<span class="token package">migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>DrupalSqlBase</span></code>, which extends <code class=" language-php">Drupal\<span class="token package">migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>SqlBase</span></code>. The new class extends <code class=" language-php">SqlBase</code> directly.</p>
<p>Any contrib or custom source plugins that currently extend <code class=" language-php">Config</code> from the <code class=" language-php">migrate_drupal</code> module should be updated to extend <code class=" language-php">ConfigEntity</code> from the <code class=" language-php">migrate</code> module. They may need further work because the properties and methods of <code class=" language-php">DrupalSqlBase</code> are no longer available.</p>

---

## [The storage of user data in localStorage for the prepopulation in anonymous forms is disabled](https://www.drupal.org/node/3498836)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>With issue <a href="https://www.drupal.org/project/drupal/issues/749748" rel="nofollow">#749748</a> (see also <a href="https://www.drupal.org/node/2243627" rel="nofollow">change record</a>), it was introduced that personal data is stored in the browser's localStorage for anonymous user forms. In other forms (comments, message, register), this data is automatically used for anonymous users.</p>
<p>This behavior has changed for data protection reasons and the data is no longer stored in the browser and used in core forms.</p>
<p>The behavior (triggered by the attribute <code class=" language-php">data<span class="token operator">-</span>user<span class="token operator">-</span>info<span class="token operator">-</span>from<span class="token operator">-</span>browser</code> in forms) still exists, but will not be filled or used by core and be deprecated later.</p>

---

## [SqlContentEntityStorage::deleteFromDedicatedTables() argument changed](https://www.drupal.org/node/3518611)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The protected method <code class=" language-php">deleteFromDedicatedTables</code> in <code class=" language-php">SqlContentEnt‎ityStorage</code> class accepts  an array of entity ID. Previously, the parameter was a single entity. </p>
<p>This allows deleting several entities with a single SQL statement, which is more performant than a statement per entity.</p>
<p>If a custom entity storage handler is overriding or calling this method, it will need to be updated to work with the parameter change.</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-protected">protected</span> <span class="token keyword keyword-function">function</span> <span class="token function">deleteFromDedicatedTables</span><span class="token punctuation">(</span>ContentEntityInterface <span class="token variable">$entity</span><span class="token punctuation">)</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-protected">protected</span> <span class="token keyword keyword-function">function</span> <span class="token function">deleteFromDedicatedTables</span><span class="token punctuation">(</span><span class="token keyword keyword-array">array</span> <span class="token variable">$ids</span><span class="token punctuation">)</span>
</code></pre>

---

## [template_preprocess_HOOK are defined as callbacks in the theme hook](https://www.drupal.org/node/3504125)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Rather than magically named functions such as <code class=" language-php">template_preprocess_container</code> these functions are now set as callbacks on the hook_theme() defining the HOOK.</p>
<p><strong>Before (using legacy hook_theme())</strong></p>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">theme</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
  <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
    <span class="token string">'block'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
      <span class="token string">'render element'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'element'</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">,</span>
  <span class="token punctuation">]</span><span class="token punctuation">;</span>

<span class="token keyword keyword-function">function</span> <span class="token function">template_preprocess_block</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>

<span class="token punctuation">}</span>
</code></pre><p>
<strong>After, using a new OOP hook:</strong></p>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>block<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token keyword keyword-class">class</span> <span class="token class-name">ThemeHooks</span> <span class="token punctuation">{</span>

  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'theme'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">theme</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
      <span class="token string">'block'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token string">'render element'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'element'</span><span class="token punctuation">,</span>
        <span class="token string">'initial preprocess'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token scope"><span class="token keyword keyword-static">static</span><span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span> <span class="token class-name"><span class="token punctuation">.</span></span> <span class="token string">':preprocessBlock'</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">preprocessBlock</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>

  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>

</code></pre><p>
Callbacks are resolved using the <a href="https://www.drupal.org/node/3368504" rel="nofollow">CallableResolver</a>, the above syntax will be resolved as the auto-registered service (which is registered with the class name as service name) and supports non-static methods and dependency injection.</p>
<p>Note that only strings and array callback definitions that can be serialized and stored are supported. And while functions as callbacks are supported, this is discouraged as module files will eventually be deprecated. The recommended approach is to place preprocess functions either in a ThemeHooks or similarly named hook class next to the hook_theme() implementation or a dedicated service or class with static methods.</p>
<p>All template_preprocess_HOOK() callbacks in core are being converted to the new system. These callbacks are not expected to be called directly even if core occasionally calls them directly, they are not an API. Consider using template suggestions or a base hook definition instead. Minimal BC will be provided for core template_preprocess_hook functions. The legacy callbacks will be removed in Drupal 12.0.<br>
The concept of template_preprocess callbacks may only be removed in Drupal 13 (to be determined).</p>
<h3>Provide backwards compatibility with earlier Drupal core versions</h3>
<p>Discovery of legacy template preprocess function is automatically skipped for theme definitions that include an initial preprocess callback. Those legacy functions can be kept as-is or changed to call the new callback:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-function">function</span> <span class="token function">template_preprocess_block</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token scope">ThemeHooks<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">preprocessBlock</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><h3>Breaking changes</h3>
<p>These preprocess callbacks are no longer in the same preprocess functions array as regular preprocess function. It is still possible to manually add it to a new theme definition added in hook_theme_registry_alter() but this is discouraged. Code that changes or removes such templates preprocess callbacks will need to be adjusted.</p>
<p>Discovery of legacy template_preprocess_HOOK() functions is deprecated and will be removed in Drupal 12 or 13 (@todo: Update when decided)</p>

---

## [Entity form modes 'description' field can no longer be an empty string](https://www.drupal.org/node/3452144)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>In order to not have a <code class=" language-php">description</code> in a form mode the <code class=" language-php">description</code> field must be set to NULL. Previously, an empty string was allowed.</p>
<h4>Before</h4>
<pre class="codeblock language-php"><code class=" language-php">langcode<span class="token punctuation">:</span> en
status<span class="token punctuation">:</span> <span class="token boolean">true</span>
dependencies<span class="token punctuation">:</span>
  module<span class="token punctuation">:</span>
    <span class="token operator">-</span> user
id<span class="token punctuation">:</span> user<span class="token punctuation">.</span>register
label<span class="token punctuation">:</span> Register
description<span class="token punctuation">:</span><span class="token string">''</span>
targetEntityType<span class="token punctuation">:</span> user
cache<span class="token punctuation">:</span> <span class="token boolean">true</span>
</code></pre><p>This is <code class=" language-php">core<span class="token punctuation">.</span>entity_form_mode<span class="token punctuation">.</span>user<span class="token punctuation">.</span>register<span class="token punctuation">.</span>yml</code> before the change.</p>
<h4>After</h4>
<pre class="codeblock language-php"><code class=" language-php">langcode<span class="token punctuation">:</span> en
status<span class="token punctuation">:</span> <span class="token boolean">true</span>
dependencies<span class="token punctuation">:</span>
  module<span class="token punctuation">:</span>
    <span class="token operator">-</span> user
id<span class="token punctuation">:</span> user<span class="token punctuation">.</span>register
label<span class="token punctuation">:</span> Register
description<span class="token punctuation">:</span> <span class="token keyword keyword-null">null</span>
targetEntityType<span class="token punctuation">:</span> user
cache<span class="token punctuation">:</span> <span class="token boolean">true</span>
</code></pre><p>This is <code class=" language-php">core<span class="token punctuation">.</span>entity_form_mode<span class="token punctuation">.</span>user<span class="token punctuation">.</span>register<span class="token punctuation">.</span>yml</code> after the change.</p>
<p>If this field is NULL, <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityDisplayModeBase<span class="token punctuation">::</span></span><span class="token function">getDescription</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> will return an empty string, for backwards compatibility.</p>

---

## [\Drupal\node\NodeStorage::updateType is deprecated](https://www.drupal.org/node/3515214)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>node<span class="token punctuation">\</span>NodeStorage<span class="token punctuation">::</span></span>updateType</code> is deprecated, there is no replacement.</p>
<p>Previously, this function was called from <code class=" language-php"><span class="token scope">NodeType<span class="token punctuation">::</span></span>preSave</code> if a node type's ID changed, however this is dangerous and potentially destructive. Changing the id of an existing node type should not be supported by API functions.</p>

---

## [Constant blacklist renamed in core/misc/autocomplete.js](https://www.drupal.org/node/3472016)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The constant  <code class=" language-php">blacklist</code> is replaced with <code class=" language-php">denyList</code> in <code class=" language-php">core<span class="token operator">/</span>misc<span class="token operator">/</span>autocomplete<span class="token punctuation">.</span>js</code>.  </p>
<h4>Before</h4>
<p><code class=" language-php">blacklist</code><br>
<code class=" language-php">data<span class="token operator">-</span>autocomplete<span class="token operator">-</span>first<span class="token operator">-</span>character<span class="token operator">-</span>blacklist</code></p>
<h4>After</h4>
<p><code class=" language-php">denyList</code><br>
<code class=" language-php">data<span class="token operator">-</span>autocomplete<span class="token operator">-</span>first<span class="token operator">-</span>character<span class="token operator">-</span>denylist</code></p>

---

## [Use #type fieldset and add the fieldgroup as attribute class.](https://www.drupal.org/node/3515272)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The use of <code class=" language-php"><span class="token shell-comment comment">#type =&gt; </span><span class="token string">'fieldgroup'</span></code> in form elements to group form elements together is no more used. Use the 'fieldgroup' as a class in #attributes of form element and <code class=" language-php"><span class="token shell-comment comment">#type =&gt; </span><span class="token string">'fieldset'</span></code>.<br>
Also need to add the <code class=" language-php">core<span class="token operator">/</span>drupal<span class="token punctuation">.</span>fieldgroup</code> library where #type fieldgroup was used in the form.</p>
<p>Before:</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$form</span><span class="token punctuation">[</span><span class="token string">'site_information'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
      <span class="token string">'#type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'fieldgroup'</span><span class="token punctuation">,</span>
      <span class="token string">'#title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Site information'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token string">'#access'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$install_state</span><span class="token punctuation">[</span><span class="token string">'config_install_path'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre><p>
After:</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$form</span><span class="token punctuation">[</span><span class="token string">'#attached'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'library'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'core/drupal.fieldgroup'</span><span class="token punctuation">;</span>

    <span class="token variable">$form</span><span class="token punctuation">[</span><span class="token string">'site_information'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
      <span class="token string">'#type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'fieldset'</span><span class="token punctuation">,</span>
      <span class="token string">'#attributes'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span><span class="token string">'class'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span><span class="token string">'fieldgroup'</span><span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token string">'#title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Site information'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token string">'#access'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$install_state</span><span class="token punctuation">[</span><span class="token string">'config_install_path'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre>

---

## [The 'system.file.path' config key is deprecated](https://www.drupal.org/node/3517074)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The 'system.file.path' config key is deprecated in drupal:11.2.0 and will be removed from drupal 12.0.0. Use 'file_temp_path' key in settings.php instead. </p>
<p>This key was deprecated in Drupal 8.8, but the config schema was never updated.</p>
<p>See [#3039255].</p>

---

## [Unpublished nodes are no longer hidden on Content overview page when a node access module is enabled](https://www.drupal.org/node/3455665)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p><strong>Before</strong></p>
<p>The Content overview page (located at <code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>content</code>) and any other Views that utilize the "Published status or admin user" filter would always display:</p>
<ul>
<li>Published nodes</li>
<li>Unpublished nodes if a user:</li>
<ul>
<li>Is the author of the node and has the "View own unpublished content" permission, or</li>
<li>Has the "Bypass content access control" permission, or</li>
<li>Has the "View any unpublished content" permission</li>
</ul>
</ul>
<p>This behavior was consistent even when a node access module implementing <code class=" language-php"><span class="token function">hook_node_grants</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> was enabled on the site. This caused issues where unpublished nodes remained hidden in these views even when the node access module granted view access to them.</p>
<p><strong>After</strong></p>
<p>The Content overview page (located at <code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>content</code>) and any other Views that utilize the "Published status or admin user" filter will now behave differently based on the presence of a node access module:</p>
<ul>
<li>If no node access module is enabled, the behavior remains unchanged.</li>
<li>If a node access module is enabled then the "Published status or admin user" filter does not alter the query anymore. The node access module takes over the access control and responsible for granting access to published and unpublished nodes.</li>
</ul>

---

## ["Published status or admin user" Views filter becomes inactive when a node access module is enabled](https://www.drupal.org/node/3472976)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>As a solution for the issue described in <a href="/node/3449181" rel="nofollow">The Content overview page filters out unpublished nodes when a node access module is enabled</a>, the behavior of the "Published status or admin user" Views filter has been modified. Now, the filter becomes inactive when a node access module is enabled.</p>
<p>This change ensures that the node access module takes over the responsibility for access control, managing access to both published and unpublished nodes. This is necessary because the node access module is designed to handle all aspects of access control, making the additional filtering redundant and potentially conflicting.</p>
<p>Site builders can now see in the filter's configuration UI whether the "Published status or admin user" filter is active or inactive. If the filter is inactive, the UI will also display a list of node access modules that are enabled on the site and have caused the filter to become inactive.</p>
<p>The filter is active because there is no node access module is enabled on the site:<br>
<img src="/files/issues/2024-08-01/3449181-with-one-access-module.png" alt="The filter is active text"></p>
<p>The filter is inactive because one or more node access modules are enabled on the site:<br>
<img src="/files/issues/2024-08-01/3449181-with-multiple-access-modules.png" alt="The filter is inactive text"></p>

---

## [\Drupal\node\NodeViewsData() constructor requires \Drupal::service('extension.list.module')](https://www.drupal.org/node/3493129)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Calling <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>node<span class="token punctuation">\</span>NodeViewsData</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>  now require <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'extension.list.module'</span></code> in their constructor.</p>
<p>Any code initializing either of these classes should pass an instance of <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'extension.list.module'</span><span class="token punctuation">)</span></code> as an argument.</p>

---

## [Option added to SectionStorage attribute to make inline block creation optional](https://www.drupal.org/node/3485595)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>In <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/2957425" title="Status: Closed (fixed)">#2957425: Allow the inline creation of non-reusable Custom Blocks in the layout builder</a></span> a link to create inline blocks in Layout Builder was introduced to allow editors adding new non-reusable inline blocks to layout builder sections.</p>
<p>However, that logic could not be disabled, so any module providing section storages was forced to have that link, or override the "Layout Builder Choose Block" controller.</p>
<p>In order to make it optional per section storage plugin, a new optional parameter in <code class=" language-php">SectionStorage</code> annotation/attribute has been introduced: <code class=" language-php">allow_inline_blocks</code>.</p>
<p>If TRUE, the <code class=" language-php">Create content block</code> link will be shown as part of the choose block off-canvas dialog. If FALSE, the link will be hidden and will not be possible to add new inline blocks from the Layout Builder UI. Its default value is TRUE to maintain BC compatibility.</p>

---

## [StatementBase abstract class introduced](https://www.drupal.org/node/3510455)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>There is quite some duplicated or nearly duplicated code in <code class=" language-php">StatementWrapperIterator</code> and <code class=" language-php">StatementPrefetchIterator</code>.</p>
<p>A new abstract <code class=" language-php">StatementBase</code> class is introduced, parent to both <code class=" language-php">StatementWrapperIterator</code> and <code class=" language-php">StatementPrefetchIterator</code> classes. The new class generalizes most of the methods so that the child classes do not have to implement them anymore. Also, helper classes to manage results of execution statements are introduced, to cover both PDO and prefetched results. This will make simpler to develop contrib/custom Statement objects.</p>
<p>No public methods were deprecated and no APIs are impacted because of this.</p>

---

## [New config action called setProperties for use on configuration entities. simpleConfigUpdate will no longer be allowed for use on configuration entities](https://www.drupal.org/node/3515543)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>It has long been a crutch that we allow the <code class=" language-php">simpleConfigUpdate</code> config action to be applied to not just simple config files, but also configuration entities.</p>
<p>We have introduced a new config action called <code class=" language-php">setProperties</code> that allows for a similar experience for recipe authors.</p>
<p>To update your recipes, just replace the old action with the new one. That said, <strong>you should continue to use <code class=" language-php">simpleConfigUpdate</code> to alter simple config</strong> (i.e., anything that is not a config entity).</p>
<p><strong>Deprecated way</strong></p>
<pre class="codeblock language-php"><code class=" language-php">config<span class="token punctuation">:</span>
  actions<span class="token punctuation">:</span>
    <span class="token shell-comment comment"># This is a config entity, since there could be more than one of them -- i.e., there could be several config items that start with `openid_connect.client`. This, therefore, is not an appropriate usage of simpleConfigUpdate.</span>
    openid_connect<span class="token punctuation">.</span>client<span class="token punctuation">.</span>okta<span class="token punctuation">:</span>
      simpleConfigUpdate<span class="token punctuation">:</span>
        label<span class="token punctuation">:</span> <span class="token string">"My label"</span>
        settings<span class="token punctuation">.</span>client_id<span class="token punctuation">:</span> my_id
</code></pre><p>
<strong>New way</strong></p>
<pre class="codeblock language-php"><code class=" language-php">config<span class="token punctuation">:</span>
  actions<span class="token punctuation">:</span>
    openid_connect<span class="token punctuation">.</span>client<span class="token punctuation">.</span>okta<span class="token punctuation">:</span>
      <span class="token shell-comment comment"># Replace simpleConfigUpdate with setProperties.</span>
      setProperties<span class="token punctuation">:</span>
        label<span class="token punctuation">:</span> <span class="token string">"My label"</span>
        <span class="token shell-comment comment"># Can also target nested properties.</span>
        settings<span class="token punctuation">.</span>client_id<span class="token punctuation">:</span> my_id
</code></pre><p>
You cannot change the ID, nor the UUID keys of any config entity files. </p>

---

## [ConfigEntity::sort() is deprecated](https://www.drupal.org/node/3511803)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token scope">ConfigEntity<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> is deprecated and will be replaced by <code class=" language-php"><span class="token scope">ConfigEntity<span class="token punctuation">::</span></span><span class="token function">sortEntities</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> which uses the new <code class=" language-php"><span class="token scope">ConfigEntity<span class="token punctuation">::</span></span><span class="token function">compare</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>.</p>
<p>Classes which inherit from the following classes should replace all calls to <code class=" language-php">usort</code>/<code class=" language-php">uasort</code> with <code class=" language-php">sort</code> callback with <code class=" language-php"><span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">sortEntities</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>:</p>
<ul>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>block<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Block<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>config_test<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>ConfigTest<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>ConfigEntityBase<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Datetime<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>DateFormat<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityDisplayModeBase<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>search<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>SearchPage<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>shortcut<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Shortcut<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>system<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Action<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
</ul>
<p>Classes which extended the <code class=" language-php"><span class="token scope">ConfigEntity<span class="token punctuation">::</span></span><span class="token function">sort</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> need to extend the <code class=" language-php"><span class="token scope">ConfigEntity<span class="token punctuation">::</span></span><span class="token function">compare</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> function.</p>

---

## [New dependency symfony/polyfill-intl-icu](https://www.drupal.org/node/3511798)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>New dependency symfony/polyfill-intl-icu is added to improve sorting. </p>
<p>If the PHP <code class=" language-php">intl</code> extension is installed, the sorting is further improved for non-english languages by their local sorting rules.</p>

---

## [SDC stylesheets are now added in the "theme" aggregate group (as opposed to "default" group) to correct CSS source order for components](https://www.drupal.org/node/3515838)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

<p>Previously Single Directory Component CSS was added to the “default” aggregate group. The new behavior is to add it to the “theme” aggregate group. </p>
<p>The net effect of this is that SDC CSS will now get lumped in with theme CSS (irregardless if the SDC is in a module or theme). </p>
<p>This fixes a CSS source order bug where SDC CSS was injected before any theme CSS (including base and layout styles). This bug was typically worked around by adding otherwise unnecessary specificity into the component’s CSS. This workaround is no longer necessary.</p>
<h3>Before</h3>
<p><img src="/files/issues/2025-03-27/sdc-before.png" alt="Screenshot of view source showing that the SDC CSS is injected way before the themes CSS"></p>
<h3>After</h3>
<p><img src="/files/issues/2025-03-27/sdc-after.png" alt="Screenshot of view source showing that the SDC CSS is now injected way before the correctly within the themes component CSS"></p>

---

## [FieldStorageAddForm is split into FieldStorageAddController and FieldStorageAddForm](https://www.drupal.org/node/3503549)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p><strong>Important:</strong><br>
Ensure your code is using <code class=" language-php">hook_field_info_entity_type_ui_definitions_alter</code> when possible. Refer to this <a href="https://www.drupal.org/node/3408700" rel="nofollow">change record</a> for an example.</p>
<p>For code directly altering the field storage form:<br>
<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>field_ui<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>FieldStorageAddForm</span></code> is now split into <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>field_ui<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>FieldStorageAddForm</span></code> and a new controller <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>field_ui<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>FieldStorageAddController</span></code>. Field creation/editing process open in modals (field selection form, combined field settings form).<br>
This means that form alter implementations may need to be updated.<br>
<strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'form_field_ui_field_storage_add_form_alter'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">formFieldUiFieldStorageAddFormAlter</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$form</span><span class="token punctuation">,</span> FormStateInterface <span class="token variable">$form_state</span><span class="token punctuation">,</span> <span class="token variable">$form_id</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> void <span class="token punctuation">{</span>
    <span class="token variable">$storage_type</span> <span class="token operator">=</span> <span class="token variable">$form_state</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getValue</span><span class="token punctuation">(</span><span class="token string">'new_storage_type'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$storage_type</span> <span class="token operator">===</span> <span class="token string">'field_my_type'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$form</span><span class="token punctuation">[</span><span class="token string">'group_field_options_wrapper'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'description_wrapper'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token string">'#type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'item'</span><span class="token punctuation">,</span> <span class="token string">'#markup'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'My description'</span><span class="token punctuation">)</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
</code></pre><p><strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'form_field_ui_field_storage_add_form_alter'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">formFieldUiFieldStorageAddFormAlter</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$form</span><span class="token punctuation">,</span> FormStateInterface <span class="token variable">$form_state</span><span class="token punctuation">,</span> <span class="token variable">$form_id</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> void <span class="token punctuation">{</span>
    <span class="token variable">$storage_type</span> <span class="token operator">=</span> <span class="token variable">$form_state</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">[</span><span class="token string">'field_type'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$storage_type</span> <span class="token operator">===</span> <span class="token string">'field_my_type'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$form</span><span class="token punctuation">[</span><span class="token string">'field_options_wrapper'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'description_wrapper'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token string">'#type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'item'</span><span class="token punctuation">,</span> <span class="token string">'#markup'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'My description'</span><span class="token punctuation">)</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
</code></pre>

---

## [The administration theme is used by default for editing and creating Node content](https://www.drupal.org/node/3514929)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The default Node create and edit forms now use the administration theme.</p>
<h4>Before</h4>
<p>The setting <code class=" language-php">use_admin_theme</code> in <code class=" language-php">node<span class="token punctuation">.</span>settings</code> defaulted to <code class=" language-php"><span class="token boolean">false</span></code>. This meant that by default Node create and edit forms used the front-end theme.</p>
<h4>After</h4>
<p>The setting <code class=" language-php">use_admin_theme</code> in <code class=" language-php">node<span class="token punctuation">.</span>settings</code> defaulted to <code class=" language-php"><span class="token boolean">true</span></code>. This means that by default Node create and edit forms use the administration theme.</p>

---

## [node_mark() is deprecated](https://www.drupal.org/node/3514189)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php"><span class="token function">node_mark</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> function was only used on non-views Node listing pages via NodeListBuilder. This coupled the node module with the history module. The history module implements its own HistoryUserTimestamp views field with the same functionality.</p>
<p>There is no replacement, marks have been removed from the default node listing.</p>

---

## [Default content can now be assigned to a specific fallback user](https://www.drupal.org/node/3513697)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Previously, when importing default content using core's <code class=" language-php">Importer</code> class, content belonging to a non-existent user would be reassigned to the first available user with an administrative role.</p>
<p>It is now possible to explicitly set which user the content is assigned to, by passing it as a parameter to <code class=" language-php"><span class="token scope">Importer<span class="token punctuation">::</span></span>importContent</code>:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$fallback_author</span> <span class="token operator">=</span> <span class="token scope">User<span class="token punctuation">::</span></span><span class="token function">load</span><span class="token punctuation">(</span><span class="token number">40</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$finder</span> <span class="token operator">=</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">Finder</span><span class="token punctuation">(</span><span class="token string">'/path/to/some/default/content'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token scope">Importer<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">importContent</span><span class="token punctuation">(</span><span class="token variable">$finder</span><span class="token punctuation">,</span> account<span class="token punctuation">:</span> <span class="token variable">$fallback_author</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
If no account is passed to <code class=" language-php">importContent</code>, it will try to use the first available user with an administrative role, same as before.</p>

---

## [New BlockPluginInterface::createPlaceholder() method](https://www.drupal.org/node/3512518)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A new ::createPlaceholder() method has been added to BlockPluginInterface:</p>
<pre class="codeblock language-php"><code class=" language-php"> <span class="token comment" spellcheck="true">/**
   * Whether to render blocks in a placeholder.
   *
   * When blocks of this type are rendered, indicate whether they should be
   * rendered in a placeholder or not. In general, blocks that attach libraries
   * and/or render entities should be placeholdered to optimize various aspects
   * of rendering performance.
   *
   * @return bool
   *   Whether to placeholder blocks of this plugin type.
   */</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">createPlaceholder</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> bool<span class="token punctuation">;</span>
</code></pre><p>
Blocks are <a href="https://www.drupal.org/docs/drupal-apis/render-api/auto-placeholdering" rel="nofollow">already placeholdered implicitly if they specify particular cache metadata</a>. This new method allows blocks to specify that they should always be placeholdered regardless of cache metadata.</p>
<p>Several core block plugins have been set to placeholder.</p>
<p>This change should immediately enable higher cache rates for some dynamic page cache items, because high-invalidation cache tags may no longer be attached to the dynamic page cache entry and instead will only affect the individual placeholder. It also allows responses served by Drupal's big pipe module to handle more of the page with big pipe.</p>

---

## [New cache prewarm API](https://www.drupal.org/node/3386853)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A new API has been added for services to tag themselves as prewarmable.</p>
<p>To do this, add the <code class=" language-php">cache_prewarmable</code> service tag, and implement <code class=" language-php">PreWarmableInterface</code>.</p>
<p>Example</p>
<pre class="codeblock language-php"><code class=" language-php">    tags<span class="token punctuation">:</span>
      <span class="token operator">-</span> <span class="token punctuation">{</span> name<span class="token punctuation">:</span> cache_prewarmable <span class="token punctuation">}</span>
</code></pre><p>
Plugin managers may also use <code class=" language-php">PreWarmablePluginManagerTrait</code> and several core plugin managers have been adapted to use it.</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token operator">+</span>  <span class="token comment" spellcheck="true">/**
+   * Implements \Drupal\Core\PreWarm\PreWarmableInterface.
+   */</span>
<span class="token operator">+</span>  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">preWarm</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
<span class="token operator">+</span>    <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getDefinitions</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token operator">+</span>  <span class="token punctuation">}</span>
</code></pre><p>
Cache prewarming should not depend on request-specific or request-derived information like the current route, language, or theme, although you could specify these explicitly in a preWarm method as long as persistent and any static caching is accounted for. This is because the context in which a cache can be prewarmed could be prior to routing or via the CLI.</p>

---

## [Calling \Drupal\Core\Cache\RefinableCacheableDependencyTrait::addCacheableDependency with an object that doesn't implement CacheableDependencyInterface is no longer supported](https://www.drupal.org/node/3232020)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<h4>Before</h4>
<p>Calling <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Cache<span class="token punctuation">\</span>RefinableCacheableDependencyTrait<span class="token punctuation">::</span></span>addCacheableDependency</code> with an object that did not implement <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Cache<span class="token punctuation">\</span>CacheableDependencyInterface</span></code> would result in the page being uncacheable (max-age 0).</p>
<p>This is rarely what the developer intended. There are other ways to mark a page as uncacheable (e.g setting max-age to 0 explicitly)</p>
<h4>After</h4>
<p>In 12.0.0, the method <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Cache<span class="token punctuation">\</span>RefinableCacheableDependencyTrait<span class="token punctuation">::</span></span>addCacheableDependency</code> will be have a type hint of <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Cache<span class="token punctuation">\</span>CacheableDependencyInterface</span></code> for the argument.<br>
<code class=" language-php"> <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">addCacheableDependency</span><span class="token punctuation">(</span>CacheableDependencyInterface <span class="token variable">$other_object</span><span class="token punctuation">)</span> <span class="token punctuation">{</span></code></p>

---

## [MigrateExecutable may fail to free up memory on hosting that has a low memory limit due to new entity LRU cache, causing more batches to run](https://www.drupal.org/node/3512407)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php">entity<span class="token punctuation">.</span>memory_cache</code> service used by the entity storage handlers to store loaded entities in memory has changed to a LRU (least recently used) cache.</p>
<p>By default, this cache keeps up to 1,000 entities in memory and then makes room for more by removing the least recently used entity from memory. We estimate that 1,000 entities will equate to roughly 200MB of memory. </p>
<h4>Migrations</h4>
<p>Sites running migrations on some (free) hosting plans with a memory limit of, say, 128MB may now get the following error.</p>
<blockquote><p>Memory usage is now @usage (@pct% of limit @limit), not enough reclaimed, starting new batch</p></blockquote>
<p>This is because the memory is no longer manually freed up whenever we do a memory usage check.</p>
<p>We recommend lowering the number of <code class=" language-php">entity<span class="token punctuation">.</span>memory_cache<span class="token punctuation">.</span>slots</code>. This limit is set using the <code class=" language-php">entity<span class="token punctuation">.</span>memory_cache<span class="token punctuation">.</span>slots</code> container parameter in core.services.yml.</p>
<p><code class=" language-php">entity<span class="token punctuation">.</span>memory_cache<span class="token punctuation">.</span>slots<span class="token punctuation">:</span> <span class="token number">1000</span></code></p>

---

## [There is a new InstallRequirementsInterface to provide install time requirements.](https://www.drupal.org/node/3492429)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>You can now provide install time requirements in a special class with the following requirements:</p>
<ol>
<li>The filename must match the classname, for example, <code class=" language-php">ModuleInstallExampleRequirements<span class="token punctuation">.</span>php</code> must contain the class <code class=" language-php">ModuleInstallExampleRequirements</code></li>
<li>It must be in the <code class=" language-php">Drupal\<span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>module</span><span class="token punctuation">&gt;</span></span></span>\<span class="token package">Install<span class="token punctuation">\</span>Requirements</span></code> namespace and located in a sub-directory of the module <code class=" language-php">src<span class="token operator">/</span>Install<span class="token operator">/</span>Requirements</code></li>
<li>It must implement <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>InstallRequirementsInterface</span><span class="token punctuation">;</span></code></li>
<li>It must have a <code class=" language-php"><span class="token keyword keyword-public">public</span> getRequirements</code> method that returns an array of requirements as specified in <code class=" language-php">InstallRequirementsInterface</code>.</li>
<li>The code must not use any other code from the module as it loaded prior to the module being installed.</li>
</ol>
<p>This works for profiles and modules.</p>
<p>These requirements will be merged with hook_requirements('install').</p>
<p>This is not a hook.</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token delimiter">&lt;?php</span>

<span class="token keyword keyword-declare">declare</span><span class="token punctuation">(</span>strict_types<span class="token operator">=</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword keyword-function">function</span> <span class="token function">module_install_example_requirements</span><span class="token punctuation">(</span><span class="token variable">$phase</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token delimiter">&lt;?php</span>

<span class="token keyword keyword-declare">declare</span><span class="token punctuation">(</span>strict_types<span class="token operator">=</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>module_install_example<span class="token punctuation">\</span>Install<span class="token punctuation">\</span>Requirements</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>InstallRequirementsInterface</span><span class="token punctuation">;</span>

<span class="token keyword keyword-class">class</span> <span class="token class-name">ModuleInstallExampleRequirements</span> <span class="token keyword keyword-implements">implements</span> <span class="token class-name">InstallRequirementsInterface</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">/**
   * {@inheritdoc}
   */</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-function">function</span> <span class="token function">getRequirements</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

<span class="token punctuation">}</span>
</code></pre>

---

## [ModuleHandler::addModule and ModuleHandler::addProfile have been deprecated](https://www.drupal.org/node/3491200)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>ModuleHandler::addModule and ModuleHandler::addProfile have been deprecated and will be removed in Drupal 12.</p>
<p>There is no direct replacement.<br>
These methods do not gather OOP hooks as OOP hooks require a container rebuild.</p>
<p>Adding modules or profiles run time is not supported.</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'module_handler'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addModule</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'module_name'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'module_handler'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addProfile</span><span class="token punctuation">(</span><span class="token string">'profile_name'</span><span class="token punctuation">,</span> <span class="token variable">$install_state</span><span class="token punctuation">[</span><span class="token string">'profiles'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'profile_name'</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getPath</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'module_installer'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">install</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'module_name'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'module_installer'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">install</span><span class="token punctuation">(</span><span class="token string">'profile_name'</span><span class="token punctuation">,</span> <span class="token variable">$install_state</span><span class="token punctuation">[</span><span class="token string">'profiles'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'profile_name'</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getPath</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [New EntityType::getBundleListCacheTags() method](https://www.drupal.org/node/3512328)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A getBundleListCacheTags() method was added to the <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityType</span></code> class.<br>
This method makes it easier to generate the ENTITY_TYPE_list:BUNDLE cache tag.</p>
<p><strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$tags</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getEntityTypeId</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">.</span> <span class="token string">'_list:'</span> <span class="token punctuation">.</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">bundle</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
<strong>Now</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$cache</span> <span class="token operator">=</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">CacheableMetadata</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$cache</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addCacheTags</span><span class="token punctuation">(</span>\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getDefinition</span><span class="token punctuation">(</span><span class="token string">'node'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getBundleListCacheTags</span><span class="token punctuation">(</span><span class="token string">'article'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
This will be invalidated when a node of type "article" is updated, removed or added.</p>

---

## [New methods added to BatchBuilder to prevent batch sets from being added more than once](https://www.drupal.org/node/3512253)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>To prevent duplicate batches from being created inadvertently in the same page request, you can use <code class=" language-php"><span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Batch<span class="token punctuation">\</span>BatchBuilder<span class="token punctuation">::</span></span><span class="token function">registerSetId</span><span class="token punctuation">(</span><span class="token punctuation">)</span> </code>and <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Batch<span class="token punctuation">\</span>BatchBuilder</span><span class="token punctuation">:</span><span class="token function">isSetIdRegistered</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> to check and see if this batch has been built before.</p>
<p>Example:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Batch<span class="token punctuation">\</span>BatchBuilder</span><span class="token punctuation">;</span>

<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>

<span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token scope">BatchBuilder<span class="token punctuation">::</span></span><span class="token function">isSetIdRegistered</span><span class="token punctuation">(</span><span class="token string">'my_unique_id'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token variable">$batch_builder</span> <span class="token operator">=</span> <span class="token punctuation">(</span><span class="token keyword keyword-new">new</span> <span class="token class-name">BatchBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">registerSetId</span><span class="token punctuation">(</span><span class="token string">'my_unique_id'</span><span class="token punctuation">)</span>
    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setTitle</span><span class="token punctuation">(</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Batch Title'</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setFinishCallback</span><span class="token punctuation">(</span><span class="token string">'batch_example_finished_callback'</span><span class="token punctuation">)</span>
    <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setInitMessage</span><span class="token punctuation">(</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'The initialization message (optional)'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token variable">$ids</span> <span class="token keyword keyword-as">as</span> <span class="token variable">$id</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$batch_builder</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addOperation</span><span class="token punctuation">(</span><span class="token string">'batch_example_callback'</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token variable">$id</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
  <span class="token function">batch_set</span><span class="token punctuation">(</span><span class="token variable">$batch_builder</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">toArray</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [Enforce strict mode on PHP sessions](https://www.drupal.org/node/3492563)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>In order to better defend against anonymous user session fixation attacks, Drupal enforces the PHP ini value <a href="https://www.php.net/manual/en/session.configuration.php#ini.session.use-strict-mode" rel="nofollow">session.use_strict_mode = 1</a>.</p>
<p>The built-in <code class=" language-php">SessionHandler</code> class has been adapted to implement <a href="https://www.php.net/manual/en/class.sessionupdatetimestamphandlerinterface.php" rel="nofollow">\SessionUpdateTimestampHandlerInterface</a> in order to make it compatible with strict session mode.</p>
<p>Contrib and custom implementations of <code class=" language-php">SessionHandler</code> should implement <code class=" language-php">\<span class="token package">SessionUpdateTimestampHandlerInterface</span></code> either by themselves or by reusing one of the following symfony base classes:</p>
<ul>
<li><a href="https://github.com/symfony/symfony/blob/7.3/src/Symfony/Component/HttpFoundation/Session/Storage/Handler/AbstractSessionHandler.php" rel="nofollow">Symfony\Component\HttpFoundation\Session\Storage\Handler\AbstractSessionHandler</a></li>
<li><a href="https://github.com/symfony/symfony/blob/7.3/src/Symfony/Component/HttpFoundation/Session/Storage/Proxy/AbstractProxy.php" rel="nofollow">Symfony\Component\HttpFoundation\Session\Storage\Proxy\AbstractProxy</a></li>
</ul>

---

## [authorize.php, the FileTransfer and Updater systems, and all related code is deprecated](https://www.drupal.org/node/3512364)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The update.module included in Drupal core used to provide a mechanism to update contributed extensions (modules and themes) directly via the web admin user interface. This system did not use <code class=" language-php">composer</code>, so any extension with external dependencies would not be correctly updated. The UI for this has been removed in Drupal 11.2.0 (see <a href="/node/3511861" rel="nofollow">The UI for updating modules and themes via the admin interface has been removed</a>), in favor of <a href="https://www.drupal.org/docs/develop/using-composer" rel="nofollow">using composer</a> or the forthcoming "Automatic Updates" feature (currently the <a href="https://www.drupal.org/project/automatic_updates" rel="nofollow">Automatic Updates</a> contributed module, hopefully soon to be included in Drupal Core as the new "Update Manager").</p>
<p>As a result, all of these methods and classes are now deprecated:</p>
<ol>
<li><code class=" language-php">core<span class="token operator">/</span>authorize<span class="token punctuation">.</span>php</code></li>
<li><code class=" language-php"><span class="token function">system_authorized_init</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">system_authorized_get_url</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">system_authorized_batch_processing_url</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">system_authorized_run</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">system_authorized_batch_process</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php">core<span class="token operator">/</span>lib<span class="token operator">/</span>Drupal<span class="token operator">/</span>Core<span class="token operator">/</span>FileTransfer<span class="token operator">/</span><span class="token operator">*</span></code></li>
<li><code class=" language-php">core<span class="token operator">/</span>lib<span class="token operator">/</span>Drupal<span class="token operator">/</span>Core<span class="token operator">/</span>Updater<span class="token operator">/</span><span class="token operator">*</span></code></li>
<li><code class=" language-php"><span class="token function">drupal_get_filetransfer_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">hook_filetransfer_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">hook_filetransfer_info_alter</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">drupal_get_updaters</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">hook_updater_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">hook_updater_info_alter</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li><code class=" language-php"><span class="token function">hook_verify_update_archive</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></li>
<li>Every method in <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>update<span class="token operator">/</span>update<span class="token punctuation">.</span>authorize<span class="token punctuation">.</span>inc</code></li>
<li>Every method in <code class=" language-php">core<span class="token operator">/</span>modules<span class="token operator">/</span>update<span class="token operator">/</span>update<span class="token punctuation">.</span>manager<span class="token punctuation">.</span>inc</code></li>
</ol>
<p>All of this code will be removed from Drupal 12.0.0. There is no direct replacement. An entirely new system will be provided once "Automatic updates" is merged into Drupal Core.</p>
<p>See also <a href="https://www.drupal.org/node/3522119" rel="nofollow">Additional 'Update Manager' deprecations</a>.</p>

---

## [Uninstalled extensions will be checked by Update Status by default](https://www.drupal.org/node/3512744)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>From 11.2.0, new installs of the Update Status module will check uninstalled extension by default, previously the default was to only check for updates of extensions that are installed.</p>
<p>This setting is configurable via the user interface and the change only affects newly installed sites.</p>

---

## [jQuery is no longer included in JavaScript aggregates](https://www.drupal.org/node/3512788)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core

### Description

<p>jQuery is no longer included in JavaScript aggregates and will instead be served as an individual file on its own. On most sites this should result in better cache hit rates for JavaScript aggregates and as a result, lower bandwidth requirements.</p>

---

## [HandlerBase::defineExtraOptions is deprecated](https://www.drupal.org/node/3486781)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>views<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>views<span class="token punctuation">\</span>HandlerBase<span class="token punctuation">::</span></span><span class="token function">defineExtraOptions</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> is deprecated.</p>
<p>This method is no longer used in Drupal Core and will be removed before Drupal 12.0.0.</p>

---

## [The UI for updating modules and themes via the admin interface has been removed](https://www.drupal.org/node/3511861)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The Update Manager UI, which allowed administrators to download and update modules and themes through the Drupal admin interface, has been removed from core. This includes the related UI components, forms, routes, and supporting code, while preserving the update status reporting functionality.</p>
<p>What is affected?</p>
<p>The following routes have been removed:</p>
<ul>
<li><code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>reports<span class="token operator">/</span>updates<span class="token operator">/</span>update</code> (<code class=" language-php">update<span class="token punctuation">.</span>report_update</code>)</li>
<li><code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>modules<span class="token operator">/</span>update</code> (<code class=" language-php">update<span class="token punctuation">.</span>module_update</code>)</li>
<li><code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>appearance<span class="token operator">/</span>update</code> (<code class=" language-php">update<span class="token punctuation">.</span>theme_update</code>)</li>
<li><code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>update<span class="token operator">/</span>ready</code> (<code class=" language-php">update<span class="token punctuation">.</span>confirmation_page</code>)</li>
</ul>
<p>The following form classes have been removed:</p>
<ul>
<li><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>update<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>UpdateManagerUpdate</span></code></li>
<li><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>update<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>UpdateReady</span></code></li>
</ul>
<p>The following route subscriber has been removed:</p>
<p><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>update<span class="token punctuation">\</span>Routing<span class="token punctuation">\</span>UpdateRouteSubscriber</span></code></p>
<p>The following utility functions has been removed:</p>
<p><code class=" language-php"><span class="token function">_update_manager_access</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and <code class=" language-php"><span class="token function">update_manager_download_batch_finished</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></p>
<p>Additionally, all UI text and help text references to the Update Manager functionality have been removed.</p>
<p>Note that the contributed <a href="https://www.drupal.org/project/issues/automatic_updates" rel="nofollow">automatic updates</a> module provides similar functionality in a more up-to-date way.</p>

---

## [editor_load() is deprecated](https://www.drupal.org/node/3509245)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Before:<br>
<code class=" language-php"><span class="token variable">$editor</span> <span class="token operator">=</span> <span class="token function">editor_load</span><span class="token punctuation">(</span><span class="token variable">$format_id</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After (procedural/static):<br>
<code class=" language-php"><span class="token variable">$editor</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">\</span>editor<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Editor<span class="token punctuation">::</span></span><span class="token function">load</span><span class="token punctuation">(</span><span class="token variable">$format_id</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After (with dependency injection):</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
  <span class="token keyword keyword-protected">protected</span> readonly EntityTypeManagerInterface <span class="token variable">$entityTypeManager</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>

    <span class="token variable">$editor</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">entityTypeManager</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token string">'editor'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">load</span><span class="token punctuation">(</span><span class="token variable">$format_id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
</code></pre>

---

## [PerformanceData getCacheTagChecksumCount and getCacheTagIsValidCount deprecated](https://www.drupal.org/node/3511149)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token scope">PerformanceData<span class="token punctuation">::</span></span><span class="token function">getCacheTagChecksumCount</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and <code class=" language-php"><span class="token scope">PerformanceData<span class="token punctuation">::</span></span><span class="token function">getCacheTagIsValidCount</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> have both been deprecated.</p>
<p>There is no direct replacement, however the <code class=" language-php"><span class="token function">getCacheTagGroupedLookups</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and <code class=" language-php"><span class="token function">getCacheTagLookupQueryCount</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> provide a more realistic reflection of how cache tag lookups impact a page and may be used instead.</p>

---

## [Added new DebugDump extension for PHPUnit](https://www.drupal.org/node/3468251)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A <code class=" language-php">DebugDump</code> PHPUnit extension is introduced to capture the output of dump() during the tests, and print it cumulatively at the end of the testrunner execution. This also adds information about where each dump() call happened.</p>
<p>The previous approach to print the outcome of dump() while running the test no longer holds in PHPUnit 10 'run in isolation' tests (which are the totality of the Kernel and Functional test). There were already changes in PHPUnit 9 that forced the change to printing to STDERR instead of STDOUT, but later issues were reported with that approach. </p>
<p>Test code and SUT code share the same standard streams. PHPUnit is 'sealing' more and more the SUT to prevent confusion between SUT and test output.</p>
<p>In order to activate the extension, add the following to your <code class=" language-php">phpunit<span class="token punctuation">.</span>xml</code> file:</p>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>extensions</span><span class="token punctuation">&gt;</span></span></span>
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span> 
    <span class="token markup"><span class="token comment" spellcheck="true">&lt;!-- Debug dump() printer. --&gt;</span></span>
    <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>bootstrap</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>Drupal\TestTools\Extension\Dump\DebugDump<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span></span>
      <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>parameter</span> <span class="token attr-name">name</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>colors<span class="token punctuation">"</span></span> <span class="token attr-name">value</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>true<span class="token punctuation">"</span></span><span class="token punctuation">/&gt;</span></span></span>
      <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>parameter</span> <span class="token attr-name">name</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>printCaller<span class="token punctuation">"</span></span> <span class="token attr-name">value</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>true<span class="token punctuation">"</span></span><span class="token punctuation">/&gt;</span></span></span>
    <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>bootstrap</span><span class="token punctuation">&gt;</span></span></span>
  <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>extensions</span><span class="token punctuation">&gt;</span></span></span>
</code></pre><p>
The extension has two parameters:</p>
<ul>
<li><em>colors</em> - indicating whether the output should use ANSI colors.</li>
<li><em>printCaller</em> - indicating whether the dump output should include the file path and the line from where it was invoked.
</li></ul>

---

## [file_get_content_headers() is deprecated](https://www.drupal.org/node/3494172)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>As of Drupal 11.x, the function file_get_content_headers() has been deprecated and will be removed in Drupal 12.x. This previously enabled developers to get the content headers for the file, but after a review of its usage, it is being replaced by a more standardized call to a method getDownloadHeaders() in the File Entity</p>
<p>Before</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$file</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token string">'file'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">load</span><span class="token punctuation">(</span><span class="token variable">$fid</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">file_get_content_headers</span><span class="token punctuation">(</span><span class="token variable">$file</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
After</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$file</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token string">'file'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">load</span><span class="token punctuation">(</span><span class="token variable">$fid</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$file</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getDownloadHeaders</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [Display classes for all description elements in form-element.html.twig](https://www.drupal.org/node/3482370)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

<p>In the template <code class=" language-php">form<span class="token operator">-</span>element<span class="token punctuation">.</span>html<span class="token punctuation">.</span>twig</code>, certain classes are added to the <code class=" language-php">div</code> element that contains the form element description. These classes were added only when <code class=" language-php">description_display</code> was set to "after" or "invisible", but not when it was set to "before". With this change, the classes will always be set.</p>
<p>The files that are changed are the core <code class=" language-php">form<span class="token operator">-</span>element<span class="token punctuation">.</span>html<span class="token punctuation">.</span>twig</code> and that template in themes <code class=" language-php">stable9</code>, <code class=" language-php">starterkit_theme</code>, and <code class=" language-php">umami</code>.</p>
<p>Before:</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div{{</span> <span class="token attr-name">description.attributes</span> <span class="token attr-name">}}</span><span class="token punctuation">&gt;</span></span></span>
      <span class="token punctuation">{</span><span class="token punctuation">{</span> description<span class="token punctuation">.</span>content <span class="token punctuation">}</span><span class="token punctuation">}</span>
    <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span></span>
</code></pre><p>
After:</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div{{</span> <span class="token attr-name">description.attributes.addClass(description_classes)</span> <span class="token attr-name">}}</span><span class="token punctuation">&gt;</span></span></span>
      <span class="token punctuation">{</span><span class="token punctuation">{</span> description<span class="token punctuation">.</span>content <span class="token punctuation">}</span><span class="token punctuation">}</span>
    <span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span></span>
</code></pre>

---

## [\Drupal\datetime_range\DateTimeRangeConstantsInterface is deprecated](https://www.drupal.org/node/3495241)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">DateTimeRangeConstantsInterface</code> is deprecated and the constants are converted to an <code class=" language-php">enum</code>. </p>
<p>Before</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$element</span><span class="token punctuation">[</span><span class="token scope">DateTimeRangeConstantsInterface<span class="token punctuation">::</span></span><span class="token constant">START_DATE</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">buildDate</span><span class="token punctuation">(</span><span class="token variable">$start_date</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$element</span><span class="token punctuation">[</span><span class="token scope">DateTimeRangeConstantsInterface<span class="token punctuation">::</span></span><span class="token constant">END_DATE</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">buildDate</span><span class="token punctuation">(</span><span class="token variable">$end_date</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><pre class="codeblock language-php"><code class=" language-php">    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
      <span class="token string">'from_to'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token scope">DateTimeRangeConstantsInterface<span class="token punctuation">::</span></span><span class="token constant">BOTH</span><span class="token punctuation">,</span>
      <span class="token string">'separator'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'-'</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre><p>
After</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$element</span><span class="token punctuation">[</span><span class="token scope">DateTimeRangeDisplayOptions<span class="token punctuation">::</span></span>StartDate<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">buildDate</span><span class="token punctuation">(</span><span class="token variable">$start_date</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$element</span><span class="token punctuation">[</span><span class="token scope">DateTimeRangeDisplayOptions<span class="token punctuation">::</span></span>EndDate<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">buildDate</span><span class="token punctuation">(</span><span class="token variable">$end_date</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><pre class="codeblock language-php"><code class=" language-php">    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
      <span class="token string">'from_to'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token scope">DateTimeRangeDisplayOptions<span class="token punctuation">::</span></span>Both<span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span><span class="token punctuation">,</span>
      <span class="token string">'separator'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'-'</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre>

---

## [A views UI option to add CSS classes to the views table element when using table formatter](https://www.drupal.org/node/3499943)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>A new UI configuration option has been added to the views table formatter style to allow CSS classes to be added to the views <code class=" language-php"><span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>table</span><span class="token punctuation">&gt;</span></span></span></code> element. This makes it easier for site builders and themers to style table variations such as row striping, bordered/borderless rows, etc. via the UI, and to specific tables outputted by views. This affects module developers who may have needed to use views style plugins to achieve the same thing.</p>
<p>The change involves an update to the database schema. If the option is not visible it may be necessary to run 'drush updb' to update the database with the schema change.</p>
<p><strong>UI:</strong><br>
<img src="/files/3045871-after-highlighted.png" alt="New CSS class input highlighted in table formatter settings UI "></p>

---

## [$entity_type_bundle_info parameter added to EntityContentBase::__construct()](https://www.drupal.org/node/3476634)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The migrate destination plugin base class \Drupal\migrate\Plugin\migrate\destination\EntityContentBase has a new parameter added to its constructor: <code class=" language-php">EntityTypeBundleInfoInterface <span class="token variable">$entity_type_bundle_info</span><span class="token punctuation">.</span></code></p>
<p>This class provides a useful base for plugins that work with content entities, and so many plugins will inherit from it. If these plugins inject services, they will have their own __construct() and create() methods. These methods will need to be changed to account for the new parameter in the parent class.</p>
<p>In Drupal 11.1 or later, the constructor of the base class will generate a deprecation notice if the new parameter is not provided. In Drupal 12.0 or later, the constructor will generate an error.</p>
<p>Before:</p>
<pre class="codeblock language-php"><code class=" language-php">MyMigrateDestinationPlugin <span class="token keyword keyword-extends">extends</span> <span class="token class-name">EntityContentBase</span> <span class="token punctuation">{</span>

  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span><span class="token keyword keyword-array">array</span> <span class="token variable">$configuration</span><span class="token punctuation">,</span> <span class="token variable">$plugin_id</span><span class="token punctuation">,</span> <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span> MigrationInterface <span class="token variable">$migration</span><span class="token punctuation">,</span> EntityStorageInterface <span class="token variable">$storage</span><span class="token punctuation">,</span> <span class="token keyword keyword-array">array</span> <span class="token variable">$bundles</span><span class="token punctuation">,</span> EntityFieldManagerInterface <span class="token variable">$entity_field_manager</span><span class="token punctuation">,</span> FieldTypePluginManagerInterface <span class="token variable">$field_type_manager</span><span class="token punctuation">,</span> MyService <span class="token variable">$my_service</span><span class="token punctuation">,</span> <span class="token operator">?</span>AccountSwitcherInterface <span class="token variable">$account_switcher</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token scope"><span class="token keyword keyword-parent">parent</span><span class="token punctuation">::</span></span><span class="token function">__construct</span><span class="token punctuation">(</span><span class="token variable">$configuration</span><span class="token punctuation">,</span> <span class="token variable">$plugin_id</span><span class="token punctuation">,</span> <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span> <span class="token variable">$migration</span><span class="token punctuation">,</span> <span class="token variable">$storage</span><span class="token punctuation">,</span> <span class="token variable">$bundles</span><span class="token punctuation">,</span> <span class="token variable">$entity_field_manager</span><span class="token punctuation">,</span> <span class="token variable">$field_type_manager</span><span class="token punctuation">,</span> <span class="token variable">$account_switcher</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">my_service</span> <span class="token operator">=</span> <span class="token variable">$my_service</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token comment" spellcheck="true">/**
   * {@inheritdoc}
   */</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-function">function</span> <span class="token function">create</span><span class="token punctuation">(</span>ContainerInterface <span class="token variable">$container</span><span class="token punctuation">,</span> <span class="token keyword keyword-array">array</span> <span class="token variable">$configuration</span><span class="token punctuation">,</span> <span class="token variable">$plugin_id</span><span class="token punctuation">,</span> <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span> <span class="token operator">?</span>MigrationInterface <span class="token variable">$migration</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$entity_type</span> <span class="token operator">=</span> <span class="token scope"><span class="token keyword keyword-static">static</span><span class="token punctuation">::</span></span><span class="token function">getEntityTypeId</span><span class="token punctuation">(</span><span class="token variable">$plugin_id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-return">return</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">static</span><span class="token punctuation">(</span>
      <span class="token variable">$configuration</span><span class="token punctuation">,</span>
      <span class="token variable">$plugin_id</span><span class="token punctuation">,</span>
      <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span>
      <span class="token variable">$migration</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_type.manager'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token function">array_keys</span><span class="token punctuation">(</span><span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_type.bundle.info'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getBundleInfo</span><span class="token punctuation">(</span><span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_field.manager'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'plugin.manager.field.field_type'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'my_service'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'account_switcher'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
    <span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

</code></pre><p>
After:</p>
<pre class="codeblock language-php"><code class=" language-php">MyMigrateDestinationPlugin <span class="token keyword keyword-extends">extends</span> <span class="token class-name">EntityContentBase</span> <span class="token punctuation">{</span>

  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span><span class="token keyword keyword-array">array</span> <span class="token variable">$configuration</span><span class="token punctuation">,</span> <span class="token variable">$plugin_id</span><span class="token punctuation">,</span> <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span> MigrationInterface <span class="token variable">$migration</span><span class="token punctuation">,</span> EntityStorageInterface <span class="token variable">$storage</span><span class="token punctuation">,</span> <span class="token keyword keyword-array">array</span> <span class="token variable">$bundles</span><span class="token punctuation">,</span> EntityFieldManagerInterface <span class="token variable">$entity_field_manager</span><span class="token punctuation">,</span> FieldTypePluginManagerInterface <span class="token variable">$field_type_manager</span><span class="token punctuation">,</span> MyService <span class="token variable">$my_service</span><span class="token punctuation">,</span> <span class="token operator">?</span>AccountSwitcherInterface <span class="token variable">$account_switcher</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">,</span> <span class="token operator">?</span>EntityTypeBundleInfoInterface <span class="token variable">$entity_type_bundle_info</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token scope"><span class="token keyword keyword-parent">parent</span><span class="token punctuation">::</span></span><span class="token function">__construct</span><span class="token punctuation">(</span><span class="token variable">$configuration</span><span class="token punctuation">,</span> <span class="token variable">$plugin_id</span><span class="token punctuation">,</span> <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span> <span class="token variable">$migration</span><span class="token punctuation">,</span> <span class="token variable">$storage</span><span class="token punctuation">,</span> <span class="token variable">$bundles</span><span class="token punctuation">,</span> <span class="token variable">$entity_field_manager</span><span class="token punctuation">,</span> <span class="token variable">$field_type_manager</span><span class="token punctuation">,</span> <span class="token variable">$account_switcher</span><span class="token punctuation">,</span> <span class="token variable">$entity_type_bundle_info</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">my_service</span> <span class="token operator">=</span> <span class="token variable">$my_service</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token comment" spellcheck="true">/**
   * {@inheritdoc}
   */</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-function">function</span> <span class="token function">create</span><span class="token punctuation">(</span>ContainerInterface <span class="token variable">$container</span><span class="token punctuation">,</span> <span class="token keyword keyword-array">array</span> <span class="token variable">$configuration</span><span class="token punctuation">,</span> <span class="token variable">$plugin_id</span><span class="token punctuation">,</span> <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span> <span class="token operator">?</span>MigrationInterface <span class="token variable">$migration</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$entity_type</span> <span class="token operator">=</span> <span class="token scope"><span class="token keyword keyword-static">static</span><span class="token punctuation">::</span></span><span class="token function">getEntityTypeId</span><span class="token punctuation">(</span><span class="token variable">$plugin_id</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-return">return</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">static</span><span class="token punctuation">(</span>
      <span class="token variable">$configuration</span><span class="token punctuation">,</span>
      <span class="token variable">$plugin_id</span><span class="token punctuation">,</span>
      <span class="token variable">$plugin_definition</span><span class="token punctuation">,</span>
      <span class="token variable">$migration</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_type.manager'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token function">array_keys</span><span class="token punctuation">(</span><span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_type.bundle.info'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getBundleInfo</span><span class="token punctuation">(</span><span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_field.manager'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'plugin.manager.field.field_type'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'my_service'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'account_switcher'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'entity_type.bundle.info'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
    <span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

</code></pre>

---

## [Classloader with support for moving/deprecating classes](https://www.drupal.org/node/3509577)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A new API and associated classloader has been added to enable renaming classes in the code base.</p>
<p>This allows classes to be renamed or moved to different namespaces, without having to leave the old class in the code base, only the services.yml entries are required.</p>
<p><code class=" language-php">module<span class="token punctuation">.</span>services<span class="token punctuation">.</span>yml</code> now supports a <code class=" language-php">moved_classes</code> parameter. This specifies the old and new class, with optional deprecation suppport.</p>
<p>Without deprecation:</p>
<pre class="codeblock language-php"><code class=" language-php">parameters<span class="token punctuation">:</span>
  core<span class="token punctuation">.</span>moved_classes<span class="token punctuation">:</span>
    <span class="token string">'Drupal\Core\StringTranslation\TranslationWrapper'</span><span class="token punctuation">:</span>
      <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> <span class="token string">'Drupal\Core\StringTranslation\TranslatableMarkup'</span>
</code></pre><p>
With deprecation:</p>
<pre class="codeblock language-php"><code class=" language-php">parameters<span class="token punctuation">:</span>
  module_autoload_test<span class="token punctuation">.</span>moved_classes<span class="token punctuation">:</span>
    <span class="token string">'Drupal\module_autoload_test\Foo'</span><span class="token punctuation">:</span>
      <span class="token keyword keyword-class">class</span><span class="token punctuation">:</span> <span class="token string">'Drupal\module_autoload_test\Bar'</span>
      deprecation_version<span class="token punctuation">:</span> drupal<span class="token punctuation">:</span><span class="token number">11.2</span><span class="token punctuation">.</span><span class="token number">0</span>
      removed_version<span class="token punctuation">:</span> drupal<span class="token punctuation">:</span><span class="token number">12.0</span><span class="token punctuation">.</span><span class="token number">0</span>
      change_record<span class="token punctuation">:</span> https<span class="token punctuation">:</span><span class="token comment" spellcheck="true">//www.drupal.org/project/drupal/issues/3502882</span>
</code></pre><p>
When the old class is autoloaded, the classloader will detect this, and swap it for the new one using class_alias(). When the deprecation-related keys are specified, the classloader will trigger <code class=" language-php"><span class="token constant">E_USER_DEPRECATED</span></code> with a message.</p>
<p>The non-deprecation version should only be used where there is a specific reason to leave the alias permanently, for example if the class is used in serialized data which cannot be easily updated.</p>

---

## [GDToolkit supports AVIF image format](https://www.drupal.org/node/3348348)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The Drupal core GDToolkit now supports the <a href="https://aomediacodec.github.io/av1-avif/" rel="nofollow">AVIF image format</a>.</p>
<p>Please note that this requires platform dependencies:</p>
<ul>
<li>It requires the PHP GD extension to be based on <code class=" language-php">libgd</code> version 2.3.2 or later including <code class=" language-php">libavif</code> support. See <a href="https://php.watch/versions/8.1/gd-avif" rel="nofollow">https://php.watch/versions/8.1/gd-avif</a> for instructions how to make sure it is added to your installation of PHP.</li>
</ul>

---

## [Usage of \PDO::FETCH_* constants to indicate fetch mode is deprecated](https://www.drupal.org/node/3488338)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>In <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3345938" title="Status: Closed (fixed)">#3345938: Deprecate support for unused \PDO::FETCH_* modes</a></span>, we normalized the fetch modes used in Drupal's database operations. However, we fell short of introducing our own logic to indicate the fetch mode, and kept using <code class=" language-php">\<span class="token scope">PDO<span class="token punctuation">::</span></span><span class="token constant">FETCH</span><span class="token operator">*</span></code> constants for the purpose.</p>
<p>Now a new <code class=" language-php">FetchAs</code> enum provides the supported modes. Usage of <code class=" language-php">\<span class="token scope">PDO<span class="token punctuation">::</span></span><span class="token constant">FETCH_</span><span class="token operator">*</span></code> constants in method calls outside of database driver code is deprecated.</p>
<p>The new <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Database<span class="token punctuation">\</span>Statement<span class="token punctuation">\</span>FetchAs</span></code> values and the <code class=" language-php">\<span class="token scope">PDO<span class="token punctuation">::</span></span><span class="token constant">FETCH_</span><span class="token operator">*</span></code> constants they are replacing:</p>
<table>
<thead>
<tr>
<th scope="col">FETCH_* constant</th>
<th scope="col">Replace with</th>
</tr>
</thead>
<tbody>
<tr class="even">
<td>\PDO::FETCH_ASSOC</td>
<td>Drupal\Core\Database\Statement\FetchAs::Associative</td>
</tr>
<tr class="odd">
<td>\PDO::FETCH_CLASS | \PDO::FETCH_PROPS_LATE</td>
<td>Drupal\Core\Database\Statement\FetchAs::ClassObject</td>
</tr>
<tr class="even">
<td>\PDO::FETCH_COLUMN</td>
<td>Drupal\Core\Database\Statement\FetchAs::Column</td>
</tr>
<tr class="odd">
<td>\PDO::FETCH_NUM</td>
<td>Drupal\Core\Database\Statement\FetchAs::List</td>
</tr>
<tr class="even">
<td>\PDO::FETCH_OBJ</td>
<td>Drupal\Core\Database\Statement\FetchAs::Object</td>
</tr>
</tbody>
</table>
<p>NOTE: An unintended side-effect of the deprecation assertion logic is to disallow the previously-allowed option to pass a class name as a string to functions like <code class=" language-php"><span class="token function">fetchAllAssoc</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>. Starting in Drupal 11.2, these need to be migrated to a separate call to <code class=" language-php"><span class="token function">setFetchMode</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>.</p>

---

## [Plugins converted from Annotations to Attributes in 11.2.0](https://www.drupal.org/node/3505424)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The following plugins now use Attributes. The change notice announcing the use of attributes for plugins is <a href="https://www.drupal.org/node/3395575" rel="nofollow">Plugin implementations should use PHP attributes instead of annotations</a>.</p>
<table>
<thead>
<tr><td>Annotation class</td>
<td>Attribute class</td>
</tr></thead>
<tbody>
<tr>
<td><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Annotation<span class="token punctuation">\</span>MigrateSource</span></code></td>
<td><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>MigrateSource</span></code></td>
</tr>
</tbody>
</table>
<h4>Exception</h4>
<p>Classes that extend <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>DrupalSqlBase</span></code> still use annotations (<code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Annotation<span class="token punctuation">\</span>MigrateSource</span></code> instead of <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>MigrateSource</span></code>).</p>
<h4>Related</h4>
<p><a href="https://www.drupal.org/node/3505422" rel="nofollow">Plugins converted from Annotations to Attributes in 11.1.0</a><br>
<a href="https://www.drupal.org/node/3229001" rel="nofollow">Plugins converted from Annotations to Attributes in 10.3.0</a></p>

---

## [Added methods to add dependencies to a migration](https://www.drupal.org/node/3471509)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Methods <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Migration<span class="token punctuation">::</span></span><span class="token function">addRequiredDependencies</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>and <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Migration<span class="token punctuation">::</span></span><span class="token function">addOptionalDependencies</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> are added to simplify adding dependencies to migration plugins.</p>
<p>Both methods have one argument: an array of migration IDs. They both return the migration object, so they can be chained with other methods.</p>
<h3>Examples</h3>
<h4>addRequiredDependencies</h4>
<p><code class=" language-php"><span class="token variable">$migration</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addRequiredDependencies</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'my_project_users'</span><span class="token punctuation">,</span> <span class="token string">'my_project_categories'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<h4>addOptionalDependencies()</h4>
<p><code class=" language-php"><span class="token variable">$migration</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addOptionalDependencies</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'my_project_block'</span><span class="token punctuation">,</span> <span class="token string">'my_project_menu'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>With these new methods, we can now simplify some code:</p>
<h4>Before Drupal 11.2.0</h4>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$migration_dependencies</span> <span class="token operator">=</span> <span class="token variable">$migration</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getMigrationDependencies</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">array_push</span><span class="token punctuation">(</span><span class="token variable">$migration_dependencies</span><span class="token punctuation">[</span><span class="token string">'required'</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getEntityTypeMigrationId</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$migration_dependencies</span><span class="token punctuation">[</span><span class="token string">'required'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token function">array_unique</span><span class="token punctuation">(</span><span class="token variable">$migration_dependencies</span><span class="token punctuation">[</span><span class="token string">'required'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$migration</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">set</span><span class="token punctuation">(</span><span class="token string">'migration_dependencies'</span><span class="token punctuation">,</span> <span class="token variable">$migration_dependencies</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><h4>Starting with Drupal 11.2.0</h4>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$migration</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addRequiredDependencies</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getEntityTypeMigrationId</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [\Drupal\menu_ui\Plugin\Menu\LocalAction\MenuLinkAdd is deprecated](https://www.drupal.org/node/3490245)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>menu_ui<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Menu<span class="token punctuation">\</span>LocalAction<span class="token punctuation">\</span>MenuLinkAdd</span></code> is deprecated and will be removed in Drupal 12. Instead, use <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Menu<span class="token punctuation">\</span>LocalActionWithDestination</span></code>.  </p>
<p>This change removes the dependency on the <code class=" language-php">menu_ui</code> module. Sites using the <code class=" language-php">MenuLinkAdd</code> plugin should check to determine if the menu_ui module can be uninstalled when using the <code class=" language-php">LocalActionWithDestination</code>.</p>

---

## [Passing NULL or an empty string to Number::alphadecimalToInt() is deprecated](https://www.drupal.org/node/3494472)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token scope">Number<span class="token punctuation">::</span></span><span class="token function">alphadecimalToInt</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> will no longer accept NULL or the empty string as they are not valid alphadecimal values.</p>
<p>In Drupal 12.0.0, the deprecation warning will be replaced with an InvalidArgumentException.</p>

---

## [New hook_entity_duplicate() and hook_ENTITY_TYPE_duplicate() hooks](https://www.drupal.org/node/3268812)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>New hooks are added that allow to control a new duplicate created by <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityInterface<span class="token punctuation">::</span></span><span class="token function">createDuplicate</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>.</p>
<p>This can be used to change the duplicate entity generated by that method and for example allows to duplicate related field instances and entity displays.</p>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-class">class</span> <span class="token class-name">EntityHooks</span> <span class="token punctuation">{</span>

  <span class="token keyword keyword-protected">protected</span> <span class="token variable">$duplicateSources</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

  <span class="token comment" spellcheck="true">/**
   * Implements hook_entity_duplicate().
   */</span>
  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'entity_duplicate'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">entityDuplicate</span><span class="token punctuation">(</span>EntityInterface <span class="token variable">$duplicate</span><span class="token punctuation">,</span> EntityInterface <span class="token variable">$entity</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> void <span class="token punctuation">{</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$duplicate</span> <span class="token keyword keyword-instanceof">instanceof</span> <span class="token class-name">SomeInterface</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$duplicate</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">set</span><span class="token punctuation">(</span><span class="token string">'name'</span><span class="token punctuation">,</span> <span class="token variable">$duplicate</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">label</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">.</span> <span class="token string">' (duplicate)'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token comment" spellcheck="true">// Store the source so it can be used in the entity_insert hook.</span>
      <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">duplicateSources</span><span class="token punctuation">[</span><span class="token variable">$duplicate</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">uuid</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token variable">$entity</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token comment" spellcheck="true">/**
   * Implements hook_entity_insert().
   */</span>
  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'entity_insert'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">entityInsert</span><span class="token punctuation">(</span>EntityInterface <span class="token variable">$entity</span><span class="token punctuation">)</span> <span class="token punctuation">:</span> void <span class="token punctuation">{</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$entity</span> <span class="token keyword keyword-instanceof">instanceof</span> <span class="token class-name">SomeInterface</span> <span class="token operator">&amp;&amp;</span> <span class="token function">isset</span><span class="token punctuation">(</span><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">duplicateSources</span><span class="token punctuation">[</span><span class="token variable">$duplicate</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">uuid</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment" spellcheck="true">// Do something with based on the duplicate source, like cloning all variations.</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre><p>
This covers at least part of the functionality offered by projects such as <a href="https://drupal.org/project/replicate" rel="nofollow">https://drupal.org/project/replicate</a>. These projects might be deprecated/updated to depend on its in favor of their own events and hooks.</p>

---

## [Workspaces listing page permission changes](https://www.drupal.org/node/3496982)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>Previously access to visit the workspaces listing page <code class=" language-php"><span class="token operator">/</span>admin<span class="token operator">/</span>config<span class="token operator">/</span>workflow<span class="token operator">/</span>workspaces</code> used the <code class=" language-php">administer workspaces</code> or <code class=" language-php">edit any workspace</code> permissions. That access now also includes the "view" permission: <code class=" language-php">view any workspace</code>.</p>

---

## [Entity storage entity_type_id_values cache tags have been removed](https://www.drupal.org/node/3505638)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>(Content) entity types share a common entity cache bin for their persistent cache tag.</p>
<p>Previously, each entity type automatically added a cache tag which allowed to invalidate the cache for just that entity type, such as node_values and user_values.</p>
<p>However, entity storage cache invalidation is transparent and a full cache clear for a specific entity type is almost never necessary except in edge cases (such as directly altering data in the tables) and tests (also rarely needed now, see <a href="https://www.drupal.org/node/3491185" rel="nofollow">https://www.drupal.org/node/3491185</a>).</p>
<p>To avoid unnecessary several additional cache tags on most page requests, these cache tags have been removed. Calling resetCache() on an entity storage without any IDs will now invalidate the full entity cache bin.</p>
<p>If a specific entity type requires semi-frequent full cache invalidation then the responsible entity storage class can still add additional cache tags.</p>
<p>See more on how and why cache tags are being optimized and reduced in the following change record: <a href="https://www.drupal.org/node/3505248" rel="nofollow">https://www.drupal.org/node/3505248</a></p>

---

## [Float and List (float) fields can no longer be added via the field UI](https://www.drupal.org/node/3505587)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>The 'Float' and 'List (float)' field types can no longer be added via field UI.</p>
<p>Configured 'Float' or 'List (float)' fields on existing sites will be unaffected.</p>
<p>The fields are also available to use via programmatically created configuration or for base and bundle fields added via the API.</p>
<p>To make them visible again, hook_field_info_alter() must be implemented in a module:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'field_info_alter'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">fieldInfoAlter</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$info</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token variable">$info</span><span class="token punctuation">[</span><span class="token string">'list_float'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'no_ui'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token constant">FALSE</span><span class="token punctuation">;</span>
  <span class="token variable">$info</span><span class="token punctuation">[</span><span class="token string">'float'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'no_ui'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token constant">FALSE</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [SqlBase::prepareQuery invoked for ::doCount and ::__toString](https://www.drupal.org/node/3502899)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The class <code class=" language-php">Drupal\<span class="token package">migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>SqlBase</span></code> is updated. This is the base class for most SQL-based migration source plugins.</p>
<p>Previously, <code class=" language-php"><span class="token scope">SqlBase<span class="token punctuation">::</span></span><span class="token function">prepareQuery</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> was not invoked from <code class=" language-php"><span class="token function">doCount</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> nor <code class=" language-php"><span class="token function">__toString</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>. This leads to counts and string representation of the query that are not actually representative of the final query. This is because <code class=" language-php">prepareQuery</code> could be adjusted for query tag alters or any other more dynamic modifications. If you already adjust for this in a custom Sql-based plugin, that is no longer necessary.</p>
<h3>Previously</h3>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__toString</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">(</span>string<span class="token punctuation">)</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token keyword keyword-protected">protected</span> <span class="token keyword keyword-function">function</span> <span class="token function">doCount</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">(</span>int<span class="token punctuation">)</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">countQuery</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">execute</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fetchField</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
</code></pre><p>
Prepare query allows folks to alter the query, which can adjust the count and string representation of the query.</p>
<h3>Now</h3>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__toString</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">(</span>string<span class="token punctuation">)</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">prepareQuery</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token keyword keyword-protected">protected</span> <span class="token keyword keyword-function">function</span> <span class="token function">doCount</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">(</span>int<span class="token punctuation">)</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">prepareQuery</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">countQuery</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">execute</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fetchField</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
</code></pre>

---

## [New multiple get methods for render and variation cache](https://www.drupal.org/node/3504958)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>New ::getMultiple() methods have been added to <code class=" language-php">VariationCache</code> and <code class=" language-php">RenderCache</code> to allow multiple render cache items to be fetched at once. These are mostly intended to be used internally by Drupal core to support the new CachedPlaceholderStrategy which immediately replaces render cache placeholders when the placeholder replacement itself is found in the cache.</p>

---

## [entity_test_create_bundle(), entity_test_delete_bundle() are deprecated and replaced with an EntityTestHelper methods](https://www.drupal.org/node/3497049)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token function">entity_test_create_bundle</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> is deprecated and replaced with the <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>entity_test<span class="token punctuation">\</span>EntityTestHelper<span class="token punctuation">::</span></span><span class="token function">createBundle</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">.</span></code><br>
<code class=" language-php"><span class="token function">entity_test_delete_bundle</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> is deprecated and replaced with the <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>entity_test<span class="token punctuation">\</span>EntityTestHelper<span class="token punctuation">::</span></span><span class="token function">deleteBundle</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">.</span></code></p>
<p>Before:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token function">entity_test_create_bundle</span><span class="token punctuation">(</span><span class="token variable">$bundle</span><span class="token punctuation">,</span> <span class="token variable">$text</span><span class="token punctuation">,</span> <span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">entity_test_delete_bundle</span><span class="token punctuation">(</span><span class="token variable">$bundle</span><span class="token punctuation">,</span> <span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
After:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>entity_test<span class="token punctuation">\</span>EntityTestHelper</span><span class="token punctuation">;</span>
<span class="token scope">EntityTestHelper<span class="token punctuation">::</span></span><span class="token function">createBundle</span><span class="token punctuation">(</span><span class="token variable">$bundle</span><span class="token punctuation">,</span> <span class="token variable">$text</span><span class="token punctuation">,</span> <span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token scope">EntityTestHelper<span class="token punctuation">::</span></span><span class="token function">deleteBundle</span><span class="token punctuation">(</span><span class="token variable">$bundle</span><span class="token punctuation">,</span> <span class="token variable">$entity_type</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [template_preprocess() removed and inlined into ThemeManager](https://www.drupal.org/node/3500806)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>template_preprocess(), together with the internal function _template_preprocess_default_variables() and template_preprocess_default_variables hook it invokes has been inlined into the ThemeManager.</p>
<p>Previously, it was discovered and added to each theme definition as the very first preprocess function, it is now automatically and always applied to any element being rendered. This makes it more reliable and reduces the size of the theme registry.</p>
<p>Nothing in regards to the functionality it provides changes within other preprocess functions and twig templates.</p>
<p>To provide backwards compatibility for calls to template_preprocess(), there is a public getDefaultTemplateVariables() method on ThemeManager, this is marked internal, neither this nor template_preprocess() must be called directly.</p>
<p>For implementations that used <code class=" language-php"><span class="token function">drupal_static_reset</span><span class="token punctuation">(</span><span class="token string">'_template_preprocess_default_variables'</span><span class="token punctuation">)</span></code> to reset statically cached default variables, <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'theme.manager'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">resetActiveTheme</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> can be called.</p>

---

## [Drupal\migrate_drupal\Plugin\migrate\source\ContentEntity been moved to the migrate module](https://www.drupal.org/node/3498916)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The following classes have been deprecated.</p>
<ul>
<li><code class=" language-php">Drupal\<span class="token package">migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>ContentEntity</span></code></li>
<li><code class=" language-php">Drupal\<span class="token package">migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>ContentEntityDeriver</span></code></li>
</ul>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token php"><span class="token delimiter">&lt;?php</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>ContentEntity</span><span class="token punctuation">;</span>
<span class="token delimiter">?&gt;</span></span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token php"><span class="token delimiter">&lt;?php</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>ContentEntity</span><span class="token punctuation">;</span>
<span class="token delimiter">?&gt;</span></span>
</code></pre><p>
The source plugin ID <code class=" language-php">content_entity</code> is not changed, so any code that uses just the plugin ID will continue to work. This includes migration plugins.</p>
<p><strong>Exception</strong>: If a site uses the <code class=" language-php">migrate</code> module but not the <code class=" language-php">migrate_drupal</code> module, and it also uses a custom or contrib module that already defines a <code class=" language-php">content_entity</code> source plugin, then that will cause a conflict.</p>
<p>Any code that uses the class (not just the plugin ID) will have to be updated.</p>

---

## [I18nQueryTrait is moved from content_translation to migrate_drupal module](https://www.drupal.org/node/3439256)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php">Drupal\<span class="token package">content_translation<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>I18nQueryTrait</span></code> is deprecated and will be removed in Drupal 12. Instead, use <code class=" language-php">Drupal\<span class="token package">migrate_drupal<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>migrate<span class="token punctuation">\</span>source<span class="token punctuation">\</span>I18nQueryTrait</span></code>.</p>
<p>The `I18nQueryTrait` in the `content_translation` module, which was deprecated in Drupal 11.2.x, has been moved to the `migrate_drupal` module. The trait is now available at `Drupal\migrate_drupal\Plugin\migrate\source\I18nQueryTrait`. It will be removed from the `content_translation` module in Drupal 12.0.0.</p>
<p>Module developers should update their code to use the new location in the `migrate_drupal` module. The functionality provided by the trait remains the same, but its location has changed to better align with migration-focused modules.</p>

---

## [CacheBackendInterface::invalidateAll() is deprecated](https://www.drupal.org/node/3500622)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>It is possible to delete a cache and it is possible to invalidate a cache. If you delete it, the deleted cache items cease to exist. That means the item cannot be rendered from the cache under any circumstances.</p>
<p>Whereas if you invalidate data you do not destroy it; you mark it as stale, or, one might say, 'past its sell by date.' When you mark a cached item as invalid, the mark  can in fact be ignored by setting $allow_invalid to TRUE.</p>
<p>It is like an item left on the supermarket shelf that is marked 'past its sell by date' but that is still available for purchase. Whereas deletion is equivalent to incinerating the item.</p>
<p>The default database implementation of cache invalidation handles it by marking all cache items as expiring in the past. Alternative cache implementation like Redis and Memcache need to separately track delete all vs invalidate all. Redis does that with an additional cache tag per bin. This makes this relatively expensive for cache bins/items that otherwise do not use cache tags.</p>
<p>invalidateAll() has no known use case compared to deleteAll() and is deprecated.</p>
<p>Note: Instead of invalidating/deleting the render cache bin, it is recommended to invalidate the deleted cache tag, as that also covers additional cache bins such as page cache and dynamic page cache.</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">// Invalidate a whole cache bin.</span>
\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">cache</span><span class="token punctuation">(</span><span class="token string">'menu'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">invalidateAll</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token comment" spellcheck="true">// Delete everything in a whole cache bin.</span>
\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">cache</span><span class="token punctuation">(</span><span class="token string">'menu'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">deleteAll</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">// Invalidate all cached rendered elements, including page and dynamic page caches.</span>
\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'cache_tags.invalidator'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">invalidateTags</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token string">'rendered'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [Block plugins can now implement CacheOptionalInterface](https://www.drupal.org/node/3516115)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Block plugins can now indicate that they are very not worth caching by implementing <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Cache<span class="token punctuation">\</span>CacheOptionalInterface</span><span class="token punctuation">.</span></code></p>
<p>In some cases, loading a cached version of a rendered block may be more expensive than just rendering the block, especially if those blocks also have many variations.</p>
<p>A good example is the language switcher block, that varies for every URL. </p>
<p>This is different from setting max-age 0, which means that the content of a block must not be cached at all. CacheOptionalInterface only applies to to separate block cache, the block may still be cached by the dynamic and anymous page caches for example.</p>
<p>A different system that uses CacheOptionalInterface are access policies, which are often also fast and do not require persistent caching.</p>
<h4>Related issues</h4>
<p><a href="https://www.drupal.org/project/drupal/issues/2232375" rel="nofollow">Support CacheOptionalInterface in BlockViewBuilder, use it in language switcher block to prevent max-age 0 bubbling</a><br>
<a href="https://www.drupal.org/project/drupal/issues/3516051" rel="nofollow">Add CacheOptionalInterface to more blocks</a></p>

---

## [Entity queries for latest revision now return the latest workspace-specific revision](https://www.drupal.org/node/3064221)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Entity queries that are looking for latest revisions of an entity type (by using <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>Query<span class="token punctuation">\</span>QueryInterface<span class="token punctuation">::</span></span><span class="token function">latestRevision</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>), are now returning the latest workspace-specific revision when running in a workspace context, for example in the <code class=" language-php">stage</code> workspace.</p>

---

## [Drupal\views\EntityViewsData ::$fieldStorageDefinitions and ::getFieldStorageDefinitions() deprecated](https://www.drupal.org/node/3240278)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token scope">Drupal<span class="token punctuation">\</span>views<span class="token punctuation">\</span>EntityViewsData<span class="token punctuation">::</span></span><span class="token variable">$fieldStorageDefinitions</span></code> and <code class=" language-php"><span class="token scope">Drupal<span class="token punctuation">\</span>views<span class="token punctuation">\</span>EntityViewsData<span class="token punctuation">::</span></span><span class="token function">getFieldStorageDefinitions</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> have both been deprecated. These were protected properties/methods and no replacement is provided. The field schema should be obtained directly from the field definition now.</p>
<p>Before:<br>
<code class=" language-php"><span class="token variable">$field_schema</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getFieldStorageDefinitions</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">[</span><span class="token variable">$field_name</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSchema</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After<br>
<code class=" language-php"><span class="token variable">$field_schema</span> <span class="token operator">=</span> <span class="token variable">$field_definition</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getFieldStorageDefinition</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSchema</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>

---

## [\Drupal\Core\Extension\ExtensionDiscovery::$fileCache is deprecated](https://www.drupal.org/node/3490431)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>If custom or contrib code extends <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>ExtensionDiscovery</span></code> and uses the <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>ExtensionDiscovery<span class="token punctuation">::</span></span><span class="token variable">$fileCache</span></code> property, it will need to change before 12.0.0 is released. </p>
<p>If the code is writing a new entry to the cache it will need to add the property to the class and instantiate the file cache object during construction as <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>ExtensionDiscovery<span class="token punctuation">::</span></span><span class="token variable">$fileCache</span></code> will no longer be set by the parent class.</p>
<p>If the code uses the cache entry written by <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Extension<span class="token punctuation">\</span>ExtensionDiscovery</span></code> then this is no longer present and you need to optimise the cache miss path. If you are reading an info.yml file you can use the  \Drupal\Core\Extension\ExtensionDiscovery::$infoParser property if it is available.</p>

---

## [Added support for route aliases and deprecation](https://www.drupal.org/node/3317784)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Examples for adding a route alias and for deprecating a route are as follows.</p>
<h4>Alias</h4>
<pre class="codeblock language-php"><code class=" language-php">example<span class="token punctuation">.</span>overview<span class="token punctuation">:</span>
  alias<span class="token punctuation">:</span> example<span class="token punctuation">.</span>index
</code></pre><h4>Deprecation</h4>
<pre class="codeblock language-php"><code class=" language-php">example<span class="token punctuation">.</span>overview<span class="token punctuation">:</span>
  deprecated<span class="token punctuation">:</span>
    package<span class="token punctuation">:</span> <span class="token string">'drupal/example'</span>
    version<span class="token punctuation">:</span> <span class="token string">'11.2'</span>
    message<span class="token punctuation">:</span> <span class="token string">'The example.overview route is deprecated in drupal:11.2.0 and is removed in drupal:12.0.0. Use the "example.overview_new." route instead.'</span>
</code></pre><p>
Symfony <a href="https://symfony.com/blog/new-in-symfony-5-4-route-aliasing" rel="nofollow">5.4 added support</a> for aliases </p>

---

## [The SAVED_DELETED constant has been deprecated](https://www.drupal.org/node/3328750)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The SAVED_DELETED constant was not used in core and it has been deprecated. </p>
<p>There is no replacement.</p>

---

## [Includes for hook_hook_info implementations have been deprecated.](https://www.drupal.org/node/3489765)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<h2>Important notice</h2>
<p>If your module <strong><em>implements</em></strong> <code class=" language-php"><span class="token function">hook_hook_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span> </code>, e.g. <code class=" language-php"><span class="token function">my_module_hook_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span> </code>, then you should <strong>not</strong> remove that function until the minimum Drupal version you require is Drupal 12.</p>
<p>If you are seeing this deprecation that means you have a file such as my_module.tokens.inc.<br>
Core provides one for tokens and on for views. Contrib has support for more files.</p>
<p>You will need to move any functions in those files out of the files and delete the files.</p>
<p>There are two main options for handling this.</p>
<h3>Short term fix</h3>
<p>Move all functions to the .module file.</p>
<h3>Longer term fix</h3>
<p>Convert any hooks to <code class=" language-php"><span class="token shell-comment comment">#[Hook]</span></code> implementations. If you need to support Drupal &lt; 11.1 then you will need the <code class=" language-php"><span class="token shell-comment comment">#[LegacyHook]</span></code> implementation as well, but move the legacy hook to the .module file.<br>
Any helper functions will need evaluation. If only one hook or hook class uses them you can move it to a method on the Hook class.<br>
If multiple Hook classes or modules consume the helper you will want to make it a service.</p>
<p>See <a href="https://www.drupal.org/node/3442349" rel="nofollow">support for object oriented hook implementations using autowired services</a> for more information. </p>
<h4>Alternative</h4>
<p>If you need to keep the file, then you will need to manually include it after Drupal 12.</p>

---

## [Added hook_runtime_requirements() and hook_runtime_requirements_alter()](https://www.drupal.org/node/3490851)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>This runs during the status report runtime check.</p>
<p>This hook is equivalent to the <code class=" language-php"><span class="token function">hook_requirements</span><span class="token punctuation">(</span><span class="token variable">$phase</span> <span class="token operator">=</span> <span class="token string">'runtime'</span><span class="token punctuation">)</span></code>.</p>
<p>This runs after <code class=" language-php"><span class="token function">hook_requirements</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>.</p>
<p>This can be implemented using <code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'runtime_requirements'</span><span class="token punctuation">)</span><span class="token punctuation">]</span></code></p>
<p>There is a <code class=" language-php"><span class="token function">hook_runtime_requirements_alter</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> that runs after all requirements checks during status report checks.</p>
<h2>Before</h2>
<pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_requirements().
 *
 * Display information about getting upload progress bars working.
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">file_requirements</span><span class="token punctuation">(</span><span class="token variable">$phase</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token variable">$requirements</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$phase</span> <span class="token operator">!=</span> <span class="token string">'runtime'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token variable">$requirements</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
  <span class="token comment" spellcheck="true">// Check the uploadprogress extension is loaded.</span>
  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token function">extension_loaded</span><span class="token punctuation">(</span><span class="token string">'uploadprogress'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$value</span> <span class="token operator">=</span> <span class="token function">t</span><span class="token punctuation">(</span>'Enabled <span class="token punctuation">(</span><span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://pecl.php.net/package/uploadprogress<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span></span><span class="token constant">PECL</span> uploadprogress<span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span></span><span class="token punctuation">)</span>'<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$description</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
  <span class="token variable">$requirements</span><span class="token punctuation">[</span><span class="token string">'file_progress'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
    <span class="token string">'title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Upload progress'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
    <span class="token string">'value'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$value</span><span class="token punctuation">,</span>
    <span class="token string">'description'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$description</span><span class="token punctuation">,</span>
  <span class="token punctuation">]</span><span class="token punctuation">;</span>

  <span class="token keyword keyword-return">return</span> <span class="token variable">$requirements</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><h2>After</h2>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>file<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * RuntimeRequirements for file.
 */</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">FileRequirementsHook</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">/**
   * Implements hook_runtime_requirements().
   */</span>
  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'runtime_requirements'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">runtime</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
    <span class="token variable">$requirements</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token comment" spellcheck="true">// Check the uploadprogress extension is loaded.</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token function">extension_loaded</span><span class="token punctuation">(</span><span class="token string">'uploadprogress'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$value</span> <span class="token operator">=</span> <span class="token function">t</span><span class="token punctuation">(</span>'Enabled <span class="token punctuation">(</span><span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>a</span> <span class="token attr-name">href</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://pecl.php.net/package/uploadprogress<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span></span><span class="token constant">PECL</span> uploadprogress<span class="token markup"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>a</span><span class="token punctuation">&gt;</span></span></span><span class="token punctuation">)</span>'<span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token variable">$description</span> <span class="token operator">=</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token variable">$requirements</span><span class="token punctuation">[</span><span class="token string">'file_progress'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
      <span class="token string">'title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Upload progress'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token string">'value'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$value</span><span class="token punctuation">,</span>
      <span class="token string">'description'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$description</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>

    <span class="token keyword keyword-return">return</span> <span class="token variable">$requirements</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [Added hook_update_requirements() and hook_update_requirements_alter()](https://www.drupal.org/node/3490852)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>This runs during the update check.</p>
<p>This hook is equivalent to the <code class=" language-php"><span class="token function">hook_requirements</span><span class="token punctuation">(</span><span class="token variable">$phase</span> <span class="token operator">=</span> <span class="token string">'update'</span><span class="token punctuation">)</span></code>.</p>
<p>This runs after <code class=" language-php"><span class="token function">hook_requirements</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>.</p>
<p>This can be implemented using <code class=" language-php"><span class="token shell-comment comment">#[Hook(</span><span class="token string">'update_requirements'</span><span class="token punctuation">)</span><span class="token punctuation">]</span></code></p>
<p>There is a <code class=" language-php"><span class="token function">hook_update_requirements_alter</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> that runs after all requirements hooks during update checks.</p>
<h2>Before</h2>
<pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_requirements().
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">google_tag_requirements</span><span class="token punctuation">(</span><span class="token variable">$phase</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token variable">$requirements</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
  <span class="token variable">$incompatible</span> <span class="token operator">=</span> <span class="token constant">FALSE</span><span class="token punctuation">;</span>

  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$phase</span> <span class="token operator">===</span> <span class="token string">'update'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$entities</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token string">'google_tag_container'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">loadMultiple</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span>\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">moduleHandler</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">moduleExists</span><span class="token punctuation">(</span><span class="token string">'google_analytics'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$ga_settings</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">config</span><span class="token punctuation">(</span><span class="token string">'google_analytics.settings'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token variable">$accounts</span> <span class="token operator">=</span> <span class="token variable">$ga_settings</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'account'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token variable">$metrics_dimensions</span> <span class="token operator">=</span> <span class="token variable">$ga_settings</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'custom.parameters'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$entities</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$accounts</span> <span class="token operator">!==</span> <span class="token string">''</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$metrics_dimensions</span> <span class="token operator">!==</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token variable">$incompatible</span> <span class="token operator">=</span> <span class="token constant">TRUE</span><span class="token punctuation">;</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$incompatible</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment" spellcheck="true">// @todo provide a drush command usage example here.</span>
      <span class="token comment" spellcheck="true">// Where users can choose between google_tag 1.x</span>
      <span class="token comment" spellcheck="true">// and google_analytics for migration.</span>
      <span class="token variable">$requirements</span><span class="token punctuation">[</span><span class="token string">'google_tag'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
        <span class="token string">'title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Google Tag'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
        <span class="token string">'description'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'In order to use Google Tag 2.x, you must decide the upgrade path between Google Tag 1.x and Google Analytics.'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
        <span class="token string">'severity'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token constant">REQUIREMENT_ERROR</span><span class="token punctuation">,</span>
        <span class="token string">'value'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'Google Tag 2.x is incompatible with Google Analytics while upgrading from 1.x.'</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
  <span class="token keyword keyword-return">return</span> <span class="token variable">$requirements</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><h2>After</h2>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>google_tag<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">/**
 * Update Requirements for google_tag.
 */</span>
<span class="token keyword keyword-class">class</span> <span class="token class-name">GoogleTagRequirementsHook</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">/**
   * Implements hook_update_requirements().
   */</span>
  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'update_requirements'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">update</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
    <span class="token variable">$requirements</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token variable">$incompatible</span> <span class="token operator">=</span> <span class="token constant">FALSE</span><span class="token punctuation">;</span>

      <span class="token variable">$entities</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token string">'google_tag_container'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">loadMultiple</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span>\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">moduleHandler</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">moduleExists</span><span class="token punctuation">(</span><span class="token string">'google_analytics'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
       <span class="token variable">$ga_settings</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">config</span><span class="token punctuation">(</span><span class="token string">'google_analytics.settings'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token variable">$accounts</span> <span class="token operator">=</span> <span class="token variable">$ga_settings</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'account'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token variable">$metrics_dimensions</span> <span class="token operator">=</span> <span class="token variable">$ga_settings</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span><span class="token string">'custom.parameters'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$entities</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$accounts</span> <span class="token operator">!==</span> <span class="token string">''</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$metrics_dimensions</span> <span class="token operator">!==</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
          <span class="token variable">$incompatible</span> <span class="token operator">=</span> <span class="token constant">TRUE</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
      <span class="token punctuation">}</span>
      <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$incompatible</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment" spellcheck="true">// @todo provide a drush command usage example here.</span>
        <span class="token comment" spellcheck="true">// Where users can choose between google_tag 1.x</span>
        <span class="token comment" spellcheck="true">// and google_analytics for migration.</span>
        <span class="token variable">$requirements</span><span class="token punctuation">[</span><span class="token string">'google_tag'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
          <span class="token string">'title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Google Tag'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
          <span class="token string">'description'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'In order to use Google Tag 2.x, you must decide the upgrade path between Google Tag 1.x and Google Analytics.'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
          <span class="token string">'severity'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token constant">REQUIREMENT_ERROR</span><span class="token punctuation">,</span>
          <span class="token string">'value'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'Google Tag 2.x is incompatible with Google Analytics while upgrading from 1.x.'</span><span class="token punctuation">,</span>
        <span class="token punctuation">]</span><span class="token punctuation">;</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token keyword keyword-return">return</span> <span class="token variable">$requirements</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [views_entity_field_label() has been deprecated](https://www.drupal.org/node/3489411)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token function">views_entity_field_label</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> has been deprecated. Use <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'entity_field.manager'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getFieldLabels</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> (introduced in version 11.2.x) instead.</p>
<p>Before</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token function">views_entity_field_label</span><span class="token punctuation">(</span><span class="token variable">$entity_type</span><span class="token punctuation">,</span> <span class="token variable">$field_name</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
After</p>
<pre class="codeblock language-php"><code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'entity_field.manager'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getFieldLabels</span><span class="token punctuation">(</span><span class="token variable">$entity_type</span><span class="token punctuation">,</span> <span class="token variable">$field_name</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
<code class=" language-php"><span class="token function">getFieldLabels</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> has been added to <code class=" language-php">EntityFieldManagerInterface</code>.</p>

---

## [node_access_test_add_field has been removed use NodeAccessTrait instead](https://www.drupal.org/node/3498345)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>node_access_test_add_field has been removed.</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token function">node_access_test_add_field</span><span class="token punctuation">(</span><span class="token scope">NodeType<span class="token punctuation">::</span></span><span class="token function">load</span><span class="token punctuation">(</span><span class="token string">'moderated_content'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Tests<span class="token punctuation">\</span>node<span class="token punctuation">\</span>Traits<span class="token punctuation">\</span>NodeAccessTrait</span><span class="token punctuation">;</span>

<span class="token keyword keyword-class">class</span> <span class="token class-name">NodeAccessTest</span> <span class="token keyword keyword-extends">extends</span> <span class="token class-name">ModerationStateTestBase</span> <span class="token punctuation">{</span>

  <span class="token keyword keyword-use">use</span> <span class="token package">NodeAccessTrait</span><span class="token punctuation">;</span>
  <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addPrivateField</span><span class="token punctuation">(</span><span class="token scope">NodeType<span class="token punctuation">::</span></span><span class="token function">load</span><span class="token punctuation">(</span><span class="token string">'moderated_content'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>

---

## [views_field_default_views_data and related helpers have been deprecated or removed.](https://www.drupal.org/node/3489502)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<h2>views_field_default_views_data</h2>
<p>Before:<br>
<code class=" language-php"><span class="token function">views_field_default_views_data</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After direct:<br>
<code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'views.field_data_provider'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">defaultFieldImplementation</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After Dependency Injection:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>views<span class="token punctuation">\</span>FieldViewsDataProvider</span><span class="token punctuation">;</span>

<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
    <span class="token keyword keyword-protected">protected</span> readonly FieldViewsDataProvider
<span class="token variable">$fieldViewsDataProvider</span><span class="token punctuation">,</span>
  <span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>

<span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">fieldViewsDataProvider</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">defaultFieldImplementation</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">)</span>
</code></pre><h2>datetime_type_field_views_data_helper</h2>
<p>Before:<br>
<code class=" language-php"><span class="token function">datetime_type_field_views_data_helper</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">,</span> <span class="token variable">$data</span><span class="token punctuation">,</span> <span class="token variable">$column_name</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After direct:<br>
<code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'datetime.views_helper'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">buildViewsData</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">,</span> <span class="token variable">$data</span><span class="token punctuation">,</span> <span class="token variable">$column_name</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After Dependency Injection:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>datetime<span class="token punctuation">\</span>DateTimeViewsHelper</span><span class="token punctuation">;</span>

<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
    <span class="token keyword keyword-protected">protected</span> readonly DateTimeViewsHelper <span class="token variable">$dateTimeViewsHelper</span><span class="token punctuation">,</span>
  <span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>

<span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">dateTimeViewsHelper</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">buildViewsData</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">,</span> <span class="token variable">$data</span><span class="token punctuation">,</span> column_name<span class="token punctuation">)</span>
</code></pre><h2>_views_field_get_entity_type_storage</h2>
<p>Before:<br>
<code class=" language-php"><span class="token function">_views_field_get_entity_type_storage</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After direct:<br>
<code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'views.views_field_default_data'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSqlStorageForField</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p>After Dependency Injection:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>views<span class="token punctuation">\</span>FieldViewsDataProvider</span><span class="token punctuation">;</span>

<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
    <span class="token keyword keyword-protected">protected</span> readonly FieldViewsDataProvider <span class="token variable">$fieldViewsDataProvider</span><span class="token punctuation">,</span>
  <span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>

<span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">fieldViewsDataProvider</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSqlStorageForField</span><span class="token punctuation">(</span><span class="token variable">$field_storage</span><span class="token punctuation">)</span>
</code></pre><h2>_content_moderation_views_data_object</h2>
<p><strong>This has been removed.</strong><br>
Before:<br>
<code class=" language-php"><span class="token function">_content_moderation_views_data_object</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></p>
<p>After:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityTypeManagerInterface</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>content_moderation<span class="token punctuation">\</span>ModerationInformation</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>content_moderation<span class="token punctuation">\</span>ViewsData</span><span class="token punctuation">;</span>

<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">__construct</span><span class="token punctuation">(</span>
    <span class="token keyword keyword-protected">protected</span> readonly EntityTypeManagerInterface <span class="token variable">$entityTypeManager</span><span class="token punctuation">,</span>
    <span class="token keyword keyword-protected">protected</span> readonly ModerationInformation <span class="token variable">$moderationInformation</span><span class="token punctuation">,</span>
  <span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>

<span class="token variable">$viewsData</span> <span class="token operator">=</span> <span class="token keyword keyword-new">new</span> <span class="token class-name">ViewsData</span><span class="token punctuation">(</span>
      <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">entityTypeManager</span><span class="token punctuation">,</span>
      <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">moderationInformation</span>
    <span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-return">return</span> <span class="token variable">$viewsData</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getViewsData</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [Added a content_top section to the navigation for programmatic additions](https://www.drupal.org/node/3487802)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The core navigation module now includes a section named <code class=" language-php">content_top</code>, which sits right above the <code class=" language-php">content</code> section. While the <code class=" language-php">content</code> section is controlled editorially via config entities, the <code class=" language-php">content_top</code> section is controlled programmatically by module developers who wish to provide integrations for this area of the navigation. Examples include environment indicator, workspaces, and Umami. To take advantage of this new <code class=" language-php">content_top</code> section, module developers can leverage two new hooks.</p>
<h2><code class=" language-php">hook_navigation_content_top</code></h2>
<p>To add content to the <code class=" language-php">content_top</code> region, implement <code class=" language-php">hook_navigation_content_top</code>. Return an array of render arrays, keyed by a unique name, as needed for your integration:</p>
<pre><pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-function">function</span> <span class="token function">my_module_navigation_content_top</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
  <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
    <span class="token string">'my_module_foo'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
      <span class="token string">'#markup'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'foo'</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token string">'my_module_bar'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
      <span class="token string">'#markup'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'bar'</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">,</span>
  <span class="token punctuation">]</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span></code></pre></pre><h2><code class=" language-php">hook_navigation_content_top_alter</code></h2>
<p>To alter the contents of the <code class=" language-php">content_top</code> region that have been provided by other modules, implement <code class=" language-php">hook_navigation_content_top_alter</code>. The hook is provided with a render array by reference for all the integrations that have been added by <code class=" language-php">hook_navigation_content_top</code> implementations.</p>
<pre><pre class="codeblock language-php"><code class=" language-php"><span class="token keyword keyword-function">function</span> <span class="token function">my_module_navigation_content_top_alter</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$content_top</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token function">unset</span><span class="token punctuation">(</span><span class="token variable">$content_top</span><span class="token punctuation">[</span><span class="token string">'navigation_foo'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token variable">$content_top</span><span class="token punctuation">[</span><span class="token string">'navigation_bar'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'#markup'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'new bar'</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span></code></pre></pre><p>
Drupal 11.1 and later support OOP/attribute-based hooks as well.</p>
<p><code class=" language-php">my_module<span class="token operator">/</span>src<span class="token operator">/</span>Hook<span class="token operator">/</span>MyModuleNavigationHooks<span class="token punctuation">.</span>php</code></p>
<pre><pre class="codeblock language-php"><code class=" language-php"><span class="token delimiter">&lt;?php</span>

<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>my_module<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Hook<span class="token punctuation">\</span>Attribute<span class="token punctuation">\</span>Hook</span><span class="token punctuation">;</span>

<span class="token keyword keyword-class">class</span> <span class="token class-name">MyModuleNavigationHooks</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">/**
   * Provide content for Navigation content_top section.
   */</span>
  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'navigation_content_top'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">navigationContentTop</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
      <span class="token string">'my_module_foo'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token string">'#markup'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'foo'</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">,</span>
      <span class="token string">'my_module_bar'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token string">'#markup'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'bar'</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token comment" spellcheck="true">/**
   * Alter the content_top section of the Navigation module.
   */</span>
  <span class="token shell-comment comment">#[Hook(</span><span class="token string">'navigation_content_top_alter'</span><span class="token punctuation">)</span><span class="token punctuation">]</span>
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">navigationContentTopAlter</span><span class="token punctuation">(</span><span class="token keyword keyword-array">array</span> <span class="token operator">&amp;</span><span class="token variable">$content_top</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
    <span class="token function">unset</span><span class="token punctuation">(</span><span class="token variable">$content_top</span><span class="token punctuation">[</span><span class="token string">'my_module_foo'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$content_top</span><span class="token punctuation">[</span><span class="token string">'my_module_bar'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'#markup'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'updated bar'</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

<span class="token punctuation">}</span>
</code></pre></pre><p>
Adjust the render array to your needs as above. This is a basic example, but it can be as complex as you like. As always, be sure to cover your cacheability metadata where applicable.</p>

---

## [Fix for Title Resolution with _raw_variables and _title_arguments](https://www.drupal.org/node/3498715)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The title resolution mechanism in TitleResolver::getTitle() was not functioning as expected when both _raw_variables and _title_arguments were defined in a route. Specifically, the method returned the static title specified in the route instead of dynamically resolving the title using the URL parameters.</p>
<p>Example<br>
Route Definition:</p>
<pre class="codeblock language-php"><code class=" language-php">custom_module<span class="token punctuation">.</span>test_route<span class="token punctuation">:</span>
  path<span class="token punctuation">:</span> <span class="token string">'/test-route/{test}/{test2}'</span>
  defaults<span class="token punctuation">:</span>
    _controller<span class="token punctuation">:</span> <span class="token string">'\Drupal\custom_module\Controller\CustomPageController::content'</span>
    _title<span class="token punctuation">:</span> <span class="token string">'Static title @test @test2'</span>
    _title_arguments<span class="token punctuation">:</span>
      <span class="token string">'@test'</span><span class="token punctuation">:</span> <span class="token string">'value'</span>
      <span class="token string">'@test2'</span><span class="token punctuation">:</span> <span class="token string">'value2'</span>
  requirements<span class="token punctuation">:</span>
    _permission<span class="token punctuation">:</span> <span class="token string">'access content'</span>
</code></pre><p>
<strong>Before</strong>:<br>
When the route /test-route/foo/bar was accessed with dynamic values foo and bar for {test} and {test2}, the title displayed was always:</p>
<p><code class=" language-php"><span class="token keyword keyword-Static">Static</span> title value value2</code><br>
The placeholders @test and @test2 were replaced with default values from _title_arguments, ignoring the dynamic parameters.</p>
<p><strong>After</strong>:<br>
When the route /test-route/foo/bar is accessed, the title now dynamically resolves as:<br>
<code class=" language-php"><span class="token keyword keyword-Static">Static</span> title foo bar</code><br>
Here, @test and @test2 are replaced with the actual URL parameter values foo and bar.</p>
<p><strong>Resolution</strong><br>
The title mechanism now prioritizes URL parameters (_raw_variables) over the default values defined in _title_arguments. This ensures that titles dynamically reflect the passed URL parameters while still using defaults if parameters are absent.</p>

---

## [StatementPrefetchIterator::fetchColumn is deprecated.](https://www.drupal.org/node/3490312)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php"><span class="token scope">StatementPrefetchIterator<span class="token punctuation">::</span></span><span class="token function">fetchColumn</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method is a leftover of earlier times when <code class=" language-php">StatementPrefetchIterator</code> was directly extending <code class=" language-php">\<span class="token package">PDOStatement</span></code>, is called nowhere in core, and has no interface definition nor correspondence in <code class=" language-php">StatementWrapperIterator</code>.</p>
<p>The method is now deprecated.</p>
<p>Use <code class=" language-php"><span class="token scope">StatementPrefetchIterator<span class="token punctuation">::</span></span><span class="token function">fetchField</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> instead, which is API and working actually the same.</p>
<p>In core, <code class=" language-php">StatementPrefetchIterator</code> is only used by the database driver for SQLite.</p>

---

## [New "Clear cache" block added](https://www.drupal.org/node/3414904)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>A new block that allows to clear caches has been added to simplify the site administrative tasks. It can be found by its display name "Clear cache".</p>
<p>Defined in <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>system<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>Block<span class="token punctuation">\</span>ClearCacheBlock</span></code>, part of the System module, with ID "system_clear_cache_block".</p>
<p><img src="/files/clear-cache-block.png" alt="Clear cache block screenshot"></p>

---

## [LOCALE_TRANSLATION_DEFAULT_SERVER_PATTERN deprecated](https://www.drupal.org/node/3488133)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><code class=" language-php"><span class="token constant">LOCALE_TRANSLATION_DEFAULT_SERVER_PATTERN</span></code> is deprecated, use <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token constant">TRANSLATION_DEFAULT_SERVER_PATTERN</span></code> instead.</p>

---

## [Static entity caches are now automatially invalidated during POST web requests in tests](https://www.drupal.org/node/3491185)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Similar to many other static caches, statically cached entities are now automatically cleared as part of the core/lib/Drupal/Core/Test/RefreshVariablesTrait.php::refreshVariables() method, which is typically called when submitting forms in web tests but can also be called explicitly.</p>
<p>This means that it is no longer required to explicitly clear static caches before reloading an entity in such tests:</p>
<p>Example from the core issue</p>
<pre class="codeblock language-php"><code class=" language-php">
     <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">drupalGet</span><span class="token punctuation">(</span><span class="token string">'comment/'</span> <span class="token punctuation">.</span> <span class="token variable">$comment</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">id</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">.</span> <span class="token string">'/edit'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
     <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">submitForm</span><span class="token punctuation">(</span><span class="token variable">$user_edit</span><span class="token punctuation">,</span> <span class="token string">'Save'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token operator">-</span>    <span class="token variable">$comment_storage</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">resetCache</span><span class="token punctuation">(</span><span class="token punctuation">[</span><span class="token variable">$comment</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">id</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
     <span class="token variable">$comment_loaded</span> <span class="token operator">=</span> <span class="token scope">Comment<span class="token punctuation">::</span></span><span class="token function">load</span><span class="token punctuation">(</span><span class="token variable">$comment</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">id</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
Existing tests that explicitly invalidate caches should continue to work and may be optimized to remove these calls once they require Drupal 11.2+.</p>
<p>This might cause failures in tests if those tests incorrectly relied on static cache masking a problem in the implementation or test code or if they explicitly assert the behavior of the static cache.</p>

---

## [Cron is no longer run by the installer](https://www.drupal.org/node/3497839)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>Cron is no longer run as part of installing Drupal. Installs that enable the automated_cron module will have cron run when they visit the frontpage.</p>
<p>If tests rely on cron being run then the test must run cron itself.</p>

---

## [DatabaseStorage::doSetIfNotExists() method visibility changed to protected](https://www.drupal.org/node/3485431)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>The <code class=" language-php"><span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>KeyValueStore<span class="token punctuation">\</span>DatabaseStorage<span class="token punctuation">::</span></span><span class="token function">doSetIfNotExists</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method visibility has been changed from <code class=" language-php"><span class="token keyword keyword-public">public</span></code> to <code class=" language-php"><span class="token keyword keyword-protected">protected</span></code> to better reflect its internal implementation nature.</p>
<p>Custom and contrib modules may need following changes is they override the method</p>
<p>If you were using <code class=" language-php"><span class="token function">doSetIfNotExists</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> directly, use the public <code class=" language-php"><span class="token function">setIfNotExists</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method instead:</p>
<p><strong>Before:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$storage</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">doSetIfNotExists</span><span class="token punctuation">(</span><span class="token string">'key'</span><span class="token punctuation">,</span> <span class="token string">'value'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
<strong>After:</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$storage</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setIfNotExists</span><span class="token punctuation">(</span><span class="token string">'key'</span><span class="token punctuation">,</span> <span class="token string">'value'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [Drupal core can now generate JSON Schemas for content entities](https://www.drupal.org/node/3424710)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Drupal core can now generate <a href="https://json-schema.org/specification" rel="nofollow">JSON Schemas</a> for content entity types.</p>
<p>The typed data, serialization and field APIs have been enhanced to allow field-level schemas to be generated based on their storage configuration.</p>
<p>All field types shipped by core now provide JSON Schemas out of the box through their default normalizers. In addition, all the core typed data plugins provide JSON Schemas as well. This means that all core fields can generate JSON Schemas for their properties out of the box. Additionally, most field types provided by contrib or custom modules will generate JSON Schemas automatically so long as they do not provide a custom normalizer or depend on non-core typed data plugins.</p>
<p>While Drupal Core now contains an API for generating these schemas for content entity types, site-wide schema generation in an end-user format (e.g., OpenAPI) is left to contrib and custom code. The <a href="https://drupal.org/project/openapi_jsonapi" rel="nofollow"><code class=" language-php">openapi_jsonapi</code></a> module uses this new functionality to provide an OpenAPI schema document for sites using JSON:API.</p>
<p><strong>Expert mode</strong></p>
<p>In a case where custom or contrib code <em>does</em> implement a custom normalizer or typed data plugins (this is rare), they can be updated to provide JSON Schemas. If a higher-level core normalizer attempts to ascertain a JSON Schema from a class which does not support it, a no-op schema containing only a comment will be generated.</p>
<p>Custom typed data plugins which provide their value through the <code class=" language-php"><span class="token scope">TypedDataInterface<span class="token punctuation">::</span></span><span class="token function">getValue</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method may simply add the new <code class=" language-php">JsonSchema</code> attribute to that method.</p>
<p>If a different method is used (see <code class=" language-php">PrimitiveDataNormalizer</code> for example, which calls <code class=" language-php"><span class="token scope">PrimitiveInterface<span class="token punctuation">::</span></span><span class="token function">getCastedValue</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>) see <code class=" language-php"><span class="token scope">PrimitiveDataNormalizer<span class="token punctuation">::</span></span><span class="token function">getNormalizationSchema</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> for an example of a more complex pattern.</p>
<p>In only very rare cases will non-core code need to configure a normalizer explicitly for JSON Schema support; see <code class=" language-php">FieldNormalizer</code> and <code class=" language-php">FieldItemNormalizer</code> if you veer into this expert territory.</p>

---

## [Drupal's JSON:API now supports adding metadata programmatically](https://www.drupal.org/node/3280569)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Some sites might want to add metadata to resources or their relations to convey implementation-specific information.</p>
<p>Before this release you could not adjust metadata without overriding internal classes. To allow developers to add metadata a new set of events has been introduced that are dispatched.</p>
<ol>
<li><code class=" language-php">CollectResourceObjectMetaEvent</code></li>
<li><code class=" language-php">CollectRelationshipMetaEvent</code></li>
</ol>
<p>Subscribers can use the following methods to adjust the metadata of the resource:</p>
<ul>
<li><code class=" language-php"><span class="token scope">CollectResourceObjectMetaEvent<span class="token punctuation">::</span></span><span class="token function">setMetaValue</span><span class="token punctuation">(</span><span class="token variable">$property</span><span class="token punctuation">,</span> <span class="token variable">$value</span><span class="token punctuation">)</span></code> to set the meta property of the resource.</li>
<li><code class=" language-php"><span class="token scope">CollectRelationshipMetaEvent<span class="token punctuation">::</span></span><span class="token function">setMetaValue</span><span class="token punctuation">(</span><span class="token variable">$property</span><span class="token punctuation">,</span> <span class="token variable">$value</span><span class="token punctuation">)</span></code> to set the meta property of the relations of a resource.</li>
</ul>
<p>These events have the related resource and context available in the event. See <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>jsonapi_test_meta_events<span class="token punctuation">\</span>EventSubscriber<span class="token punctuation">\</span>MetaEventSubscriber</span></code> for an example of how to add meta information to your resources.</p>
<h2>Example</h2>
<p>Something you might want to do with this feature is add metadata about an external service to your JSON:API response. This would be data that is related to your resource, but it not part of the data model. For this example, we need to add some external payment information on the order for the client. This data is generated based on an external service.</p>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-namespace">namespace</span> <span class="token package">Drupal<span class="token punctuation">\</span>custom_payment<span class="token punctuation">\</span>EventSubscriber</span><span class="token punctuation">;</span>

<span class="token keyword keyword-use">use</span> <span class="token package">Drupal<span class="token punctuation">\</span>jsonapi<span class="token punctuation">\</span>Events<span class="token punctuation">\</span>CollectResourceObjectMetaEvent</span><span class="token punctuation">;</span>
<span class="token keyword keyword-use">use</span> <span class="token package">Symfony<span class="token punctuation">\</span>Component<span class="token punctuation">\</span>EventDispatcher<span class="token punctuation">\</span>EventSubscriberInterface</span><span class="token punctuation">;</span>

<span class="token keyword keyword-class">class</span> <span class="token class-name">PaymentMetaEventSubscriber</span> <span class="token keyword keyword-implements">implements</span> <span class="token class-name">EventSubscriberInterface</span> <span class="token punctuation">{</span>

  <span class="token comment" spellcheck="true">/**
    * {@inheritDoc}
    */</span> 
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-function">function</span> <span class="token function">getSubscribedEvents</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
      <span class="token scope">CollectResourceObjectMetaEvent<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'addPaymentInfoToResourceObjectMeta'</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token comment" spellcheck="true">/**
    * Custom method to addd payment info to the order object.
    *
    * @param \Drupal\jsonapi\Events\CollectResourceObjectMetaEvent $event
    *   The event to add meta resources for.
    *
    * @return void
    */</span> 
  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">addPaymentInfoToResourceObjectMeta</span><span class="token punctuation">(</span>CollectResourceObjectMetaEvent <span class="token variable">$event</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$event</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getResourceObject</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getTypeName</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">!==</span> <span class="token string">'order--order'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword keyword-return">return</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>

    <span class="token variable">$event</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">setMetaValue</span><span class="token punctuation">(</span><span class="token string">'payment_iframe_url'</span><span class="token punctuation">,</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'custom_payment.payment_data'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getIframeUrl</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

<span class="token punctuation">}</span>
</code></pre>

---

## [New methods to access original (unchanged) entity during entity update added to EntityInterface, EntityBase::$original is deprecated](https://www.drupal.org/node/3295826)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Previously, <code class=" language-php"><span class="token variable">$entity</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">original</span></code> was used to access the original entity during update operations.</p>
<p>Instead, use <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>EntityInterface<span class="token punctuation">::</span></span><span class="token function">getOriginal</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>. <code class=" language-php"><span class="token scope">EntityInterface<span class="token punctuation">::</span></span><span class="token function">setOriginal</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> can be used to manually set the original entity, for example during bulk operations to speed up performance.</p>
<p>Note the slight behavior change when saving non-default revisions. Previously, the original property was always the unchanged default revision, now it is the revision that had been loaded, so when saving a new non-default revision based on a previous non-default revision or updating it, changes can be correctly detected.</p>
<p>Implementations of EntityInterface that do not extend from EntityBase will need to add the two new methods.</p>

---

## [New Schema method to execute data definition language (DDL) SQL statements](https://www.drupal.org/node/3402926)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>SQL that executes data definition language (DDL) statements should be processed through the protected <code class=" language-php"><span class="token scope">Schema<span class="token punctuation">::</span></span><span class="token function">executeDdlStatement</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method and no longer with the <code class=" language-php"><span class="token scope">Connection<span class="token punctuation">::</span></span><span class="token function">query</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method. This allows Drupal’s transaction manager to handle gracefully autocommits that occur when a DDL statement is executed while a Drupal transaction stack is active, for databases that do not support transactional DDL (MySql, Oracle, ...).</p>
<p>Within the <code class=" language-php">Schema</code> class of a database driver, </p>
<p><strong>Before</strong></p>
<p><code class=" language-php"><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">connection</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">query</span><span class="token punctuation">(</span><span class="token string">'DROP TABLE {test}'</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>
<p><strong>After</strong></p>
<p><code class=" language-php"><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">executeDdlStatement</span><span class="token punctuation">(</span><span class="token string">'DROP TABLE {test}'</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></p>

---

## [Creating a field storage in Kerneltests without the entity schema being installed is deprecated](https://www.drupal.org/node/3493981)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>In Kerneltests, creating a field storage on an entity in which the entity schema has not been installed is deprecated. </p>
<p>In Drupal 12.0.0, the deprecation warning will be replaced with a LogicException.</p>
<p>Example:</p>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token comment" spellcheck="true">// When the next line is missing the deprecation warning will be thrown.</span>
  <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">installEntitySchema</span><span class="token punctuation">(</span><span class="token string">'entity_test'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

  <span class="token comment" spellcheck="true">// Create a field storage on the entity "entity_test".</span>
  <span class="token scope">Drupal<span class="token punctuation">\</span>field<span class="token punctuation">\</span>Entity<span class="token punctuation">\</span>FieldStorageConfig<span class="token punctuation">::</span></span><span class="token function">create</span><span class="token punctuation">(</span><span class="token punctuation">[</span>
    <span class="token string">'field_name'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'field_test'</span><span class="token punctuation">,</span>
    <span class="token string">'entity_type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'entity_test'</span><span class="token punctuation">,</span>
    <span class="token string">'type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'integer'</span><span class="token punctuation">,</span>
  <span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">save</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>

---

## [\Drupal\user\Plugin\views\argument_default\CurrentUser now requires the AccountProxyInterface](https://www.drupal.org/node/3347878)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Class <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>user<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>views<span class="token punctuation">\</span>argument_default<span class="token punctuation">\</span>CurrentUser</span></code> now requires the <code class=" language-php">AccountProxyInterface</code> in its constructor.</p>
<p>Any class extending this class needs to update its constructor to match.</p>

---

## [New container_rebuild_required key in .info.yml files](https://www.drupal.org/node/3473563)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Multiple modules can now be installed without a container rebuild in between each one. This is a significant performance improvement when installing a new Drupal site locally or on CI environments.</p>
<p>Since some modules modify the container so that any changes they make need to happen immediately or depend on a module that does so, a new <code class=" language-php">container_rebuild_required</code> key has been added to .info.yml files. When this key is set to TRUE, a container rebuild will be triggered immediately before and after installing the module. If a module does not require a container rebuild immediately after installation, it can be set to FALSE or left unset to improve performance.</p>
<p>In general, you can safely set this to FALSE or leave it unset unless you are decorating or otherwise modifying services that are used within the module install process itself or requiring a dependency that does so.</p>

---

## [Change \Drupal\Core\Config\ConfigInstaller to support installing multiple modules with a single container rebuild](https://www.drupal.org/node/3492559)

- **Version**: 11.2.0
- **Branch**: 11.2.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Code that has extended <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>ConfigInstaller</span></code> or implemented <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>ConfigInstallerInterface</span></code> will need to change in order to support Drupal 11.2. Code that calls the methods is unlikely to need to change unless it wants to use the new functionality detailed below.</p>
<p>A <code class=" language-php"><span class="token variable">$mode</span></code> argument is added to <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>ConfigInstaller<span class="token punctuation">::</span></span><span class="token function">installDefaultConfig</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>. This enables installing configuration in a modules config/install, config/optional, and other installed modules config/optional directories in separate operations. A new enum <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>DefaultConfigMode</span></code> is added to support this.</p>
<p>A <code class=" language-php"><span class="token variable">$previously_checked_config</span></code> argument is added to the protected methods <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>ConfigInstaller<span class="token punctuation">::</span></span><span class="token function">findPreExistingConfiguration</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Config<span class="token punctuation">\</span>ConfigInstaller<span class="token punctuation">::</span></span><span class="token function">findDefaultConfigWithUnmetDependencies</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code>. This argument is a list of previously checked configuration, keyed by collection name.</p>
<p>See <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/config_selector/issues/3492937" title="Status: Closed (fixed)">#3492937: Add compatibility layer for Drupal 11.2</a></span> for an example of how to add a compatibility layer to a module that decorates the config installer.</p>

---

