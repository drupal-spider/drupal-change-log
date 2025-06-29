# Drupal Change Records

## [node_title_list is deprecated](https://www.drupal.org/node/3531959)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>This is a procedural function in node.module that was previously used by statistics and forum modules. It is no longer used by Drupal core.</p>
<p>To replicate the functionality, you can do the following:</p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$nodes</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getStorage</span><span class="token punctuation">(</span><span class="token string">'node'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">loadMultiple</span><span class="token punctuation">(</span><span class="token variable">$nids</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

    <span class="token variable">$items</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token variable">$nids</span> <span class="token keyword keyword-as">as</span> <span class="token variable">$nid</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token variable">$node</span> <span class="token operator">=</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token string">'entity.repository'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getTranslationFromContext</span><span class="token punctuation">(</span><span class="token variable">$nodes</span><span class="token punctuation">[</span><span class="token variable">$nid</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token variable">$item</span> <span class="token operator">=</span> <span class="token variable">$node</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">toLink</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">toRenderable</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">renderer</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">addCacheableDependency</span><span class="token punctuation">(</span><span class="token variable">$item</span><span class="token punctuation">,</span> <span class="token variable">$node</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token variable">$items</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token variable">$item</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>

    <span class="token keyword keyword-return">return</span> <span class="token punctuation">[</span>
      <span class="token string">'#theme'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'item_list__node'</span><span class="token punctuation">,</span>
      <span class="token string">'#items'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$items</span><span class="token punctuation">,</span>
      <span class="token string">'#title'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$title</span><span class="token punctuation">,</span>
      <span class="token string">'#cache'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
        <span class="token string">'tags'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> \<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">entityTypeManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getDefinition</span><span class="token punctuation">(</span><span class="token string">'node'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getListCacheTags</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre>

---

## [Widget elements can be written using object oriented approach](https://www.drupal.org/node/3532733)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p><a href="https://www.drupal.org/node/3532720" rel="nofollow">A new object-oriented API has been added for working with form and render</a> and widgets can now written with fully object oriented code without writing any arrays, without <code class=" language-php"><span class="token shell-comment comment">#</span></code>.</p>
<p>Before:</p>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">formElement</span><span class="token punctuation">(</span>FieldItemListInterface <span class="token variable">$items</span><span class="token punctuation">,</span> <span class="token variable">$delta</span><span class="token punctuation">,</span> <span class="token keyword keyword-array">array</span> <span class="token variable">$element</span><span class="token punctuation">,</span> <span class="token keyword keyword-array">array</span> <span class="token operator">&amp;</span><span class="token variable">$form</span><span class="token punctuation">,</span> FormStateInterface <span class="token variable">$form_state</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$element</span><span class="token punctuation">[</span><span class="token string">'value'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token variable">$element</span> <span class="token operator">+</span> <span class="token punctuation">[</span>
      <span class="token string">'#type'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'textfield'</span><span class="token punctuation">,</span>
      <span class="token string">'#default_value'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token variable">$items</span><span class="token punctuation">[</span><span class="token variable">$delta</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span> <span class="token operator">?</span><span class="token operator">?</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">,</span>
      <span class="token string">'#size'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSetting</span><span class="token punctuation">(</span><span class="token string">'size'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token string">'#placeholder'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSetting</span><span class="token punctuation">(</span><span class="token string">'placeholder'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token string">'#maxlength'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getFieldSetting</span><span class="token punctuation">(</span><span class="token string">'max_length'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
      <span class="token string">'#attributes'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span><span class="token string">'class'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span><span class="token string">'js-text-full'</span><span class="token punctuation">,</span> <span class="token string">'text-full'</span><span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">,</span>
    <span class="token punctuation">]</span><span class="token punctuation">;</span>

    <span class="token keyword keyword-return">return</span> <span class="token variable">$element</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
</code></pre><p>
After:</p>
<pre class="codeblock language-php"><code class=" language-php">  <span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">singleElementObject</span><span class="token punctuation">(</span>FieldItemListInterface <span class="token variable">$items</span><span class="token punctuation">,</span> <span class="token variable">$delta</span><span class="token punctuation">,</span> Widget <span class="token variable">$widget</span><span class="token punctuation">,</span> ElementInterface <span class="token variable">$form</span><span class="token punctuation">,</span> FormStateInterface <span class="token variable">$form_state</span><span class="token punctuation">)</span><span class="token punctuation">:</span> ElementInterface <span class="token punctuation">{</span>
    <span class="token variable">$value</span> <span class="token operator">=</span> <span class="token variable">$widget</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">createChild</span><span class="token punctuation">(</span><span class="token string">'value'</span><span class="token punctuation">,</span> <span class="token scope">Textfield<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">,</span> copyProperties<span class="token punctuation">:</span> <span class="token constant">TRUE</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$value</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">default_value</span> <span class="token operator">=</span> <span class="token variable">$items</span><span class="token punctuation">[</span><span class="token variable">$delta</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span> <span class="token operator">?</span><span class="token operator">?</span> <span class="token keyword keyword-NULL">NULL</span><span class="token punctuation">;</span>
    <span class="token variable">$value</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">size</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSetting</span><span class="token punctuation">(</span><span class="token string">'size'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$value</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">placeholder</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getSetting</span><span class="token punctuation">(</span><span class="token string">'placeholder'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$value</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">maxlength</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getFieldSetting</span><span class="token punctuation">(</span><span class="token string">'max_length'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token variable">$value</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">attributes</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token string">'class'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span><span class="token string">'js-text-full'</span><span class="token punctuation">,</span> <span class="token string">'text-full'</span><span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-return">return</span> <span class="token variable">$widget</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
</code></pre><p>
Note this does not call <code class=" language-php"><span class="token scope"><span class="token keyword keyword-parent">parent</span><span class="token punctuation">::</span></span><span class="token function">singleElementObject</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and that's correct. <code class=" language-php"><span class="token scope">WidgetBase<span class="token punctuation">::</span></span>singleElementObject</code> provides backwards compatibility by calling <code class=" language-php"><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">formElement</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> and wrapping it for the modern OOP elements API, there's no need to call it once the conversion above has been done.</p>

---

## [New Object oriented approach for working with form/render arrays](https://www.drupal.org/node/3532720)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A new object-oriented API has been added for working with form and render elements.</p>
<p>The API builds on top of existing Render/Form element plugins and supports casting to and from an object.</p>
<p>The objects are provided as a convenience for working with render arrays and provide IDE integration so that supported keys can be auto-completed.</p>
<p>Once the form/render arrays have been modified using the new API, in the first instance they must be cast back to render arrays to continue to work with Drupal. Future releases will build support for these objects as first class returns.</p>
<h2>Examples<br>
</h2><h2>
</h2><p>In all the examples below <code class=" language-php"><span class="token variable">$elementInfoManager</span></code> is the element info plugin manager.<br>
This can be retrieved from the service container using <code class=" language-php">\<span class="token scope">Drupal<span class="token punctuation">::</span></span><span class="token function">service</span><span class="token punctuation">(</span><span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Render<span class="token punctuation">\</span>ElementInfoManagerInterface<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">)</span></code> or alternatively injected using available dependency injection approaches.<br>
If you're working inside a class that extends from <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Form<span class="token punctuation">\</span>FormBase</span></code>, this is also available as <code class=" language-php"><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">elementInfoManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code></p>
<h3>Create a new submit button</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$submit</span> <span class="token operator">=</span> <span class="token variable">$elementInfoManager</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fromClass</span><span class="token punctuation">(</span>\<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Render<span class="token punctuation">\</span>Element<span class="token punctuation">\</span>Submit<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token variable">$submit</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Submit'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
If you're using an IDE, you should get auto-complete suggestions on the properties you can use on the object</p>
<p><img src="/files/issues/2025-05-22/2025-05-22%2005%2018%2043.png" alt="Screenshot showing the value and submit properties being auto-completed for a submit element"></p>
<h3>Alter an existing form</h3>
<p>Adding a new child element.</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$form_obj</span> <span class="token operator">=</span> <span class="token variable">$elementInfoManager</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fromRenderable</span><span class="token punctuation">(</span><span class="token variable">$form</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$checkbox</span>  <span class="token operator">=</span> <span class="token variable">$form_obj</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">createChild</span><span class="token punctuation">(</span><span class="token string">'field_name'</span><span class="token punctuation">,</span> \<span class="token scope">Drupal<span class="token punctuation">\</span>Core<span class="token punctuation">\</span>Render<span class="token punctuation">\</span>Element<span class="token punctuation">\</span>Checkbox<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$checkbox</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">title</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Do you like this new API?'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$checkbox</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">required</span> <span class="token operator">=</span> <span class="token constant">TRUE</span><span class="token punctuation">;</span>
</code></pre><p>
Removing a child element</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$form_obj</span> <span class="token operator">=</span> <span class="token variable">$elementInfoManager</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fromRenderable</span><span class="token punctuation">(</span><span class="token variable">$form</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token comment" spellcheck="true">// No soup for you.</span>
<span class="token variable">$form_obj</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">removeChild</span><span class="token punctuation">(</span><span class="token string">'soup'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><h3>Iterating over children</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$form_obj</span> <span class="token operator">=</span> <span class="token variable">$elementInfoManager</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fromRenderable</span><span class="token punctuation">(</span><span class="token variable">$form</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$titles</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
<span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token variable">$form_obj</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getChildren</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword keyword-as">as</span> <span class="token variable">$child</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// Children are objects too.</span>
  <span class="token variable">$titles</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token variable">$child</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">title</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><h3>Getting a single child</h3>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$form_obj</span> <span class="token operator">=</span> <span class="token variable">$elementInfoManager</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fromRenderable</span><span class="token punctuation">(</span><span class="token variable">$form</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$form_obj</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">getChild</span><span class="token punctuation">(</span><span class="token string">'submit'</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Save'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><h3>Casting back to a render array</h3>
<p>At this point in time, form build methods and the like still expect a render array.<br>
So the new objects are only for developer convenience while building and working with form elements.<br>
To cast the object back to a render array to be returned from a form builder function, use the <code class=" language-php">toRenderable</code> method</p>
<pre class="codeblock language-php"><code class=" language-php">
<span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">buildForm</span><span class="token punctuation">(</span><span class="token keyword keyword-array">array</span> <span class="token variable">$form</span><span class="token punctuation">,</span> FormStateInterface <span class="token variable">$form_state</span><span class="token punctuation">)</span><span class="token punctuation">:</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">{</span>
  <span class="token variable">$form_obj</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">elementInfoManager</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">fromRenderable</span><span class="token punctuation">(</span><span class="token variable">$form</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token variable">$submit</span> <span class="token operator">=</span>  <span class="token variable">$form_obj</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">createChild</span><span class="token punctuation">(</span><span class="token string">'submit'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token variable">$submit</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">value</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">t</span><span class="token punctuation">(</span><span class="token string">'Submit'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token keyword keyword-return">return</span> <span class="token variable">$form_obj</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">toRenderable</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

</code></pre><h2>Adding support for custom element plugins</h2>
<p>Adding support to a custom/contrib element plugin is a matter of ensuring that any custom # keys are documented as <code class=" language-php">@property</code> keys on the plugin class.</p>
<p>See <a href="https://git.drupalcode.org/project/drupal/-/blob/17ea568b9a3fa8602f7cf2f69edfcee54edd2dab/core/lib/Drupal/Core/Entity/Element/EntityAutocomplete.php#L23" rel="nofollow">an example</a> from core</p>

---

## [system/base split into more conditionally loaded libraries](https://www.drupal.org/node/3530832)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Various CSS files previously included in the system/base library have been split into smaller asset libraries and are now conditionally loaded with the elements or templates that require them.</p>
<p>Themes or modules relying on CSS specified in those libraries may need to include the new library in #attached.</p>
<p>Themes that were overriding the files may need to adjust to override the new libraries.</p>
<p>See <a href="https://www.drupal.org/node/3432346" rel="nofollow">https://www.drupal.org/node/3432346</a> for examples of the changes needed.</p>
<p>Libraries moved in 11.3.0:</p>
<ul>
<li><span class="project-issue-issue-link project-issue-status-info project-issue-status-2"><a href="https://www.drupal.org/project/drupal/issues/3512285" title="Status: Fixed">#3512285: Split item-list.module.css out to its own library</a></span></li>
</ul>

---

## [A new database driver (mysqli) for MySQL/MariaDB for parallel queries](https://www.drupal.org/node/3516913)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Site builders, administrators, editors

### Description

<p>A new experimental database driver for MySQL and MariaDB has been added. The new database driver is named MySQLi. The database driver is labeled as <code class=" language-php">experimental</code> and therefore not yet fully supported. Use the database driver only for evaluating purposes. The database driver is also marked as hidden.</p>
<p>The new database driver uses another PHP extension to connect to MySQL/MariaDB. The default database driver MySQL/MariaDB uses the mysql PDO PHP extension. The new database driver uses the mysqli PHP extension. This is a more modern PHP extension which allows database queries to be run in parallel instead of sequential as with the PDO extension. For this to work we will use the async Revolt (<a href="https://revolt.run/" rel="nofollow">https://revolt.run/</a>) PHP event loop. The creation of this new database driver will unblock work on the use of the async PHP event loop. Examples of how this will make Drupal faster ():<br>
 - The loading of Drupal entities with many fields will be faster. Every field stored its values in its own database table.<br>
 - The loading of views with linked entities will be faster.<br>
 - Any place where multiple queries can run in parallel can be made faster.</p>
<p>How to test this new database driver. The database driver cannot be selected during the install process. Select the regular database driver for MySQL/MariaDB during the site install process or do a <code class=" language-php">drush site<span class="token punctuation">:</span>install</code>. When you have a running Drupal site, first install/enable the mysqli module, then change in the <code class=" language-php">settings<span class="token punctuation">.</span>php</code> file the used database connection in the following way.</p>
<p>Install the mysqli module:<br>
<code class=" language-php">drush pm<span class="token punctuation">:</span>install mysqli</code></p>
<p>Before:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$databases</span><span class="token punctuation">[</span><span class="token string">'default'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'default'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">(</span>
  <span class="token string">'database'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span>
  <span class="token string">'username'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span>
  <span class="token string">'password'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span>
  <span class="token string">'host'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span> 
  <span class="token string">'prefix'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">''</span><span class="token punctuation">,</span>
  <span class="token string">'driver'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'mysql'</span><span class="token punctuation">,</span>
  <span class="token string">'namespace'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'Drupal\\mysql\\Driver\\Database\\mysql'</span><span class="token punctuation">,</span>
  <span class="token string">'autoload'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'core/modules/mysql/src/Driver/Database/mysql/'</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
After:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$databases</span><span class="token punctuation">[</span><span class="token string">'default'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'default'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token keyword keyword-array">array</span> <span class="token punctuation">(</span>
  <span class="token string">'database'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span>
  <span class="token string">'username'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span>
  <span class="token string">'password'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span>
  <span class="token string">'host'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'db'</span><span class="token punctuation">,</span> 
  <span class="token string">'prefix'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">''</span><span class="token punctuation">,</span>
  <span class="token string">'driver'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'mysqli'</span><span class="token punctuation">,</span>
  <span class="token string">'namespace'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'Drupal\\mysqli\\Driver\\Database\\mysqli'</span><span class="token punctuation">,</span>
  <span class="token string">'autoload'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'core/modules/mysqli/src/Driver/Database/mysqli/'</span><span class="token punctuation">,</span>
  <span class="token string">'dependencies'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token keyword keyword-array">array</span><span class="token punctuation">(</span>
    <span class="token string">'mysql'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token keyword keyword-array">array</span><span class="token punctuation">(</span>
      <span class="token string">'namespace'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'Drupal\\mysql'</span><span class="token punctuation">,</span>
      <span class="token string">'autoload'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'core/modules/mysql/src/'</span><span class="token punctuation">,</span>
    <span class="token punctuation">)</span><span class="token punctuation">,</span>
  <span class="token punctuation">)</span><span class="token punctuation">,</span>
<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><p>
On the status page in the Database section you should see something like: "MySQL via mysqli" or "MariaDB via mysqli".</p>
<p>The long term goal is for this new database driver to replace the existing mysql database driver. </p>

---

## [ConfigurableTrait and ConfigurablePluginBase available to reduce plugin boilerplate](https://www.drupal.org/node/2853355)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>A majority of plugins implement <code class=" language-php">\<span class="token package">Drupal<span class="token punctuation">\</span>Component<span class="token punctuation">\</span>Plugin<span class="token punctuation">\</span>ConfigurableInterface</span></code>, and each have to provide the getter and setter methods.</p>
<p>Due to the existence of <code class=" language-php"><span class="token keyword keyword-public">public</span> <span class="token keyword keyword-function">function</span> <span class="token function">defaultConfiguration</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code>, they must also handle merging defaults in with explicit configuration.</p>
<p>To reduce the boilerplate, there is now  \Drupal\Core\Plugin\ConfigurablePluginBase class as well as \Drupal\Core\Plugin\ConfigurableTrait.</p>
<p>The trait uses NestedArray::MergeDeepArray to merge the provided configuration with the defaults, which properly merges nested keys while preserving integer keys.</p>
<p>Where possible, a plugin should extend ConfigurablePluginBase to implement ConfigurableInterface. This will properly merge the plugin defaults with the provided configuration in the constructor, and provide the getConfiguration and setConfiguration methods. It will also provide a defaultConfiguration method which returns an empty array.  Plugins should override this method with their own defaults.</p>
<p>For plugins that must extend a different plugin base class (and when the base class itself can not extend ConfigurablePluginBase) a trait is also provided to provide the getConfiguration, setConfiguration, and defaultConfiguration methods provided by the interface. If using the trait instead of the base class, it is important to call -&gt;setConfiguration in the class constructor after calling the parent constructor in order to properly merge the provided configuration with the defaults. see <a href="https://www.drupal.org/node/2852190" rel="nofollow">https://www.drupal.org/node/2852190</a> for more detail.</p>
<p>The following plugin base classes have been converted to use the new boilerplate:</p>
<p>ConfigurableActionBase<br>
VariantBase<br>
SelectionPluginBase<br>
LayoutDefault<br>
ImageEffectBase<br>
ConfigurableSearchPluginBase<br>
WorkflowTypeBase</p>
<p>Of those above, All except for ImageEffectBase and WorkflowTypeBase have had their merging logic converted to use mergeDeepArray from either mergeDeep or simple array addition.  These should be functionally equivalent, but may return the array keys in a slightly different order, which might require test updates in contrib.</p>

---

## [Transaction::commitOrRelease() method introduced to explicity commit a transaction](https://www.drupal.org/node/3512006)

- **Version**: 11.3.0
- **Branch**: 11.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>Drupal by default commits a transaction when its Transaction object goes out of scope, during the Transaction object destruction.</p>
<p>This is a pattern that deviates from normal PHP behaviour, where explicit commit is required and without explicit commit an uncompleted transaction is rolled back. Also, recently this behaviour caused problems when the destruction order of objects is not predictable (see <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3405976" title="Status: Closed (fixed)">#3405976: Transaction autocommit during shutdown relies on unreliable object destruction order (xdebug 3.3+ enabled)</a></span>).</p>
<p>A new <code class=" language-php"><span class="token scope">Transaction<span class="token punctuation">::</span></span><span class="token function">commitOrRelease</span><span class="token punctuation">(</span><span class="token punctuation">)</span></code> method is added to the Transaction object, to mean <strong>explicit</strong> commit/savepoint release.</p>
<p><em>::commitOrRelease()</em> indicates that the transaction control is returned to the parent level in a nested transaction scenario like Drupal's - e.g. a 'savepoint' transaction object returns control to its parent 'root' transaction (that can still be rolled back entirely if necessary); a 'root' transaction returns control to the database by committing (=persisting changed data) the db transaction, etc.  </p>
<p><strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token keyword keyword-try">try</span> <span class="token punctuation">{</span>
      <span class="token variable">$transaction</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">connection</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">startTransaction</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">insertValues</span> <span class="token keyword keyword-as">as</span> <span class="token variable">$insert_values</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token variable">$stmt</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">execute</span><span class="token punctuation">(</span><span class="token variable">$insert_values</span><span class="token punctuation">,</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">queryOptions</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token keyword keyword-catch">catch</span> <span class="token punctuation">(</span><span class="token class-name"><span class="token punctuation">\</span>Exception</span> <span class="token variable">$e</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token function">isset</span><span class="token punctuation">(</span><span class="token variable">$transaction</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token comment" spellcheck="true">// One of the INSERTs failed, rollback the whole batch.</span>
        <span class="token variable">$transaction</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">rollBack</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token punctuation">}</span>
      <span class="token comment" spellcheck="true">// Rethrow the exception for the calling code.</span>
      <span class="token keyword keyword-throw">throw</span> <span class="token variable">$e</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre><p>
<strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php">    <span class="token variable">$transaction</span> <span class="token operator">=</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">connection</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">startTransaction</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-try">try</span> <span class="token punctuation">{</span>
      <span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">insertValues</span> <span class="token keyword keyword-as">as</span> <span class="token variable">$insert_values</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token variable">$stmt</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">execute</span><span class="token punctuation">(</span><span class="token variable">$insert_values</span><span class="token punctuation">,</span> <span class="token this">$this</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token property">queryOptions</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
      <span class="token punctuation">}</span>
      <span class="token variable">$transaction</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">commitOrRelease</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token keyword keyword-catch">catch</span> <span class="token punctuation">(</span><span class="token class-name"><span class="token punctuation">\</span>Exception</span> <span class="token variable">$e</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment" spellcheck="true">// One of the INSERTs failed, rollback the whole batch and rethrow the exception for the calling code.</span>
      <span class="token variable">$transaction</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">rollBack</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
      <span class="token keyword keyword-throw">throw</span> <span class="token variable">$e</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
</code></pre>

---

## [Experimental Symfony Mailer Module](https://www.drupal.org/node/3519253)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Module developers

### Description

<p>An experimental mailer module has been added which puts the necessary services in place such that bleeding-edge contrib and custom code can start using the mail delivery part of <a href="https://symfony.com/doc/current/mailer.html" rel="nofollow">Symfony Mailer</a> by simply retrieving / referencing a configured mailer service from / in the container.</p>
<p>The new mail API is subject to breaking changes. This change record will be updated with every iteration.</p>
<h3>Setup</h3>
<p>Enable the experimental <code class=" language-php">mailer</code> module. Then configure the transport <a href="https://symfony.com/doc/current/mailer.html#using-built-in-transports" rel="nofollow">DSN</a> by specifying its components in the <code class=" language-php">mailer_dsn</code> config. The following examples may serve as a starting point:</p>
<ul>
<li>For the default sendmail transport:<br>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$config</span><span class="token punctuation">[</span><span class="token string">'system.mail'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'mailer_dsn'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
  <span class="token string">'scheme'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'sendmail'</span><span class="token punctuation">,</span>
  <span class="token string">'host'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'default'</span><span class="token punctuation">,</span>
<span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre></li>
<li>For mailpit on localhost:<br>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$config</span><span class="token punctuation">[</span><span class="token string">'system.mail'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'mailer_dsn'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
  <span class="token string">'scheme'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'smtp'</span><span class="token punctuation">,</span>
  <span class="token string">'host'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'localhost'</span><span class="token punctuation">,</span>
  <span class="token string">'port'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token number">1025</span><span class="token punctuation">,</span>
<span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre></li>
<li>For authenticated SMTP:<br>
<pre class="codeblock language-php"><code class=" language-php"><span class="token variable">$config</span><span class="token punctuation">[</span><span class="token string">'system.mail'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'mailer_dsn'</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token punctuation">[</span>
  <span class="token string">'scheme'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'smtp'</span><span class="token punctuation">,</span>
  <span class="token string">'host'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'smtp.example.com'</span><span class="token punctuation">,</span>
  <span class="token string">'user'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'some-username@example.com'</span><span class="token punctuation">,</span>
  <span class="token string">'password'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'correct horse battery staple'</span><span class="token punctuation">,</span>
  <span class="token string">'options'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">[</span>
    <span class="token string">'local_domain'</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token string">'example.org'</span><span class="token punctuation">,</span>
  <span class="token punctuation">]</span><span class="token punctuation">,</span>
<span class="token punctuation">]</span><span class="token punctuation">;</span>
</code></pre></li>
</ul>
<h3>Preliminary Mail Delivery API</h3>
<p>See issue <span class="project-issue-issue-link project-issue-status-info project-issue-status-7"><a href="https://www.drupal.org/project/drupal/issues/3379794" title="Status: Closed (fixed)">#3379794: Add symfony mailer transports to Dependency Injection Container (mail delivery layer)</a></span>.</p>
<ul>
<li>Service: <code class=" language-php">Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>MailerInterface</span></code>:<br>
Custom and contrib modules may use this service to pass mails to the mail delivery layer. This is the main entry point for the mail delivery layer.</li>

<li>Service: <code class=" language-php">Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>TransportInterface</span></code>:<br>
Custom and contrib modules may use this service to directly inject mails to the configured mail transport for delivery. This will skip Symfony messenger (if configured). This should only be necessary in advanced use cases.</li>
<li>Service: <code class=" language-php">Drupal\<span class="token package">Core<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>TransportServiceFactoryInterface</span></code>:<br>
Custom and contrib modules may decorate or replace this service in order to customize construction of <code class=" language-php">Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>TransportInterface</span></code>. This should only be necessary in advanced use cases. E.g., if certain messages are sent via dedicated transports.</li>
<li>Events <a href="https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/Mailer/Event/MessageEvent.php" rel="nofollow">MessageEvent</a>, <a href="https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/Mailer/Event/SentMessageEvent.php" rel="nofollow">SentMessageEvent</a> and <a href="https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/Mailer/Event/FailedMessageEvent.php" rel="nofollow">FailedMessageEvent</a>:<br>
Custom and contrib modules may register event subscribers to act on emails before and after they are sent.</li>
<li>Abstract service <code class=" language-php">Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>AbstractTransportFactory</span></code> and service tag <code class=" language-php">mailer<span class="token punctuation">.</span>transport_factory</code>:<br>
Custom and contrib modules may supply third-party or custom transport factories using <code class=" language-php">Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>AbstractTransportFactory</span></code> as their parent service, tagged with <code class=" language-php">mailer<span class="token punctuation">.</span>transport_factory</code> and typically with <code class=" language-php">Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>AbstractTransportFactory</span></code> as their parent class.</li>
<li>The <code class=" language-php">mailer_sendmail_commands</code> setting:<br>
An array of command lines which are allowed as the <code class=" language-php">command</code> option in the sendmail transport.</li>
</ul>
<h4>Example: Send a Message</h4>
<p>The following example can serve as a starting point for experiments with the Symfony Mailer mail delivery API:</p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">// Get the mailer instance from the container.</span>
<span class="token variable">$mailer</span> <span class="token operator">=</span> <span class="token variable">$container</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">get</span><span class="token punctuation">(</span>\<span class="token scope">Symfony<span class="token punctuation">\</span>Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>MailerInterface<span class="token punctuation">::</span></span><span class="token keyword keyword-class">class</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment" spellcheck="true">// Create a new message.</span>
<span class="token variable">$email</span> <span class="token operator">=</span> <span class="token keyword keyword-new">new</span> <span class="token class-name"><span class="token punctuation">\</span>Symfony<span class="token punctuation">\</span>Component<span class="token punctuation">\</span>Mime<span class="token punctuation">\</span>Email</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token variable">$email</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">subject</span><span class="token punctuation">(</span><span class="token string">"Test message"</span><span class="token punctuation">)</span>
  <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">from</span><span class="token punctuation">(</span><span class="token string">'test@localhost.localdomain'</span><span class="token punctuation">)</span>
  <span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">text</span><span class="token punctuation">(</span><span class="token string">'Hello test runner!'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword keyword-try">try</span> <span class="token punctuation">{</span>
  <span class="token variable">$mailer</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">send</span><span class="token punctuation">(</span><span class="token variable">$email</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">to</span><span class="token punctuation">(</span><span class="token string">'admin@localhost.localdomain'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token comment" spellcheck="true">// $messenger-&gt;addStatus($this-&gt;t('Sent test message'));</span>
<span class="token punctuation">}</span>
<span class="token keyword keyword-catch">catch</span> <span class="token punctuation">(</span><span class="token class-name">RuntimeException</span> <span class="token variable">$e</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// $messenger-&gt;addError($this-&gt;t('Failed to send test message'));</span>
<span class="token punctuation">}</span>
</code></pre><h4>Example: Register a Third-Party Transport</h4>
<p>Contrib and custom modules providing third-party transports supply their own service tagged with the <code class=" language-php">mailer<span class="token punctuation">.</span>transport_factory</code> tag, deriving from the abstract transport class:</p>
<pre class="codeblock language-php"><code class=" language-php">services<span class="token punctuation">:</span>
  <span class="token shell-comment comment"># Example of third-party service integration, requires symfony/google-mailer</span>
  <span class="token shell-comment comment"># https:</span><span class="token comment" spellcheck="true">//packagist.org/packages/symfony/google-mailer</span>
  Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Bridge<span class="token punctuation">\</span>Google<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>GmailTransportFactory</span><span class="token punctuation">:</span>
    <span class="token keyword keyword-parent">parent</span><span class="token punctuation">:</span> Symfony\<span class="token package">Component<span class="token punctuation">\</span>Mailer<span class="token punctuation">\</span>Transport<span class="token punctuation">\</span>AbstractTransportFactory</span>
    tags<span class="token punctuation">:</span>
      <span class="token operator">-</span> <span class="token punctuation">{</span> name<span class="token punctuation">:</span> mailer<span class="token punctuation">.</span>transport_factory <span class="token punctuation">}</span>
</code></pre><h3>Preliminary Mail Building API</h3>
<p>Not designed/implemented yet.</p>

---

## [Olivero's table.css moved to a standalone library and attached only to tables.](https://www.drupal.org/node/3517675)

- **Version**: 11.3.0
- **Branch**: 11.3.x
- **Status**: Published (View all published change records)
- **Project**: Drupal core
- **Impacts**: Themers

### Description

<p>The <code class=" language-php">table<span class="token punctuation">.</span>css</code> file from the Olivero theme has moved from the base global-styling library to a new standalone library <code class=" language-php">olivero<span class="token punctuation">.</span>table</code>. </p>
<p>This library is now specifically attached in the following contexts:</p>
<ul>
<li>The <code class=" language-php">table<span class="token punctuation">.</span>html<span class="token punctuation">.</span>twig</code> template</li>
<li>Views rendered with the <code class=" language-php">views<span class="token operator">-</span>view<span class="token operator">-</span>table</code> template</li>
<li>Fields of the types: text_with_summary, text, and text_long to support tables added through CKEditor.</li>
</ul>
<p>Example:<br>
<strong>Before</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_preprocess_HOOK().
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">olivero_preprocess_field</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token variable">$rich_field_types</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token string">'text_with_summary'</span><span class="token punctuation">,</span> <span class="token string">'text'</span><span class="token punctuation">,</span> <span class="token string">'text_long'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token function">in_array</span><span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'field_type'</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token variable">$rich_field_types</span><span class="token punctuation">,</span> <span class="token constant">TRUE</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'class'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'text-content'</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'field_type'</span><span class="token punctuation">]</span> <span class="token operator">==</span> <span class="token string">'image'</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'element'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'#view_mode'</span><span class="token punctuation">]</span> <span class="token operator">==</span> <span class="token string">'full'</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">"element"</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">"#is_multiple"</span><span class="token punctuation">]</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'field_name'</span><span class="token punctuation">]</span> <span class="token operator">!==</span> <span class="token string">'user_picture'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'class'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'wide-content'</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span></code></pre><pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_preprocess_table().
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">olivero_preprocess_table</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// Mark the whole table and the first cells if rows are draggable.</span>
  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'rows'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$draggable_row_found</span> <span class="token operator">=</span> <span class="token constant">FALSE</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'rows'</span><span class="token punctuation">]</span> <span class="token keyword keyword-as">as</span> <span class="token operator">&amp;</span><span class="token variable">$row</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment" spellcheck="true">/** @var \Drupal\Core\Template\Attribute $row['attributes'] */</span>
      <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$row</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$row</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasClass</span><span class="token punctuation">(</span><span class="token string">'draggable'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token variable">$draggable_row_found</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
          <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'class'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'draggable-table'</span><span class="token punctuation">;</span>
          <span class="token variable">$draggable_row_found</span> <span class="token operator">=</span> <span class="token constant">TRUE</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span></code></pre><p>
<strong>After</strong></p>
<pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_preprocess_HOOK().
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">olivero_preprocess_field</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token variable">$rich_field_types</span> <span class="token operator">=</span> <span class="token punctuation">[</span><span class="token string">'text_with_summary'</span><span class="token punctuation">,</span> <span class="token string">'text'</span><span class="token punctuation">,</span> <span class="token string">'text_long'</span><span class="token punctuation">]</span><span class="token punctuation">;</span>

  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token function">in_array</span><span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'field_type'</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token variable">$rich_field_types</span><span class="token punctuation">,</span> <span class="token constant">TRUE</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'class'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'text-content'</span><span class="token punctuation">;</span>
    <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'#attached'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'library'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'olivero/olivero.table'</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'field_type'</span><span class="token punctuation">]</span> <span class="token operator">==</span> <span class="token string">'image'</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'element'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'#view_mode'</span><span class="token punctuation">]</span> <span class="token operator">==</span> <span class="token string">'full'</span> <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">"element"</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">"#is_multiple"</span><span class="token punctuation">]</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'field_name'</span><span class="token punctuation">]</span> <span class="token operator">!==</span> <span class="token string">'user_picture'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'class'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'wide-content'</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span></code></pre><pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_preprocess_table().
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">olivero_preprocess_table</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token comment" spellcheck="true">// Mark the whole table and the first cells if rows are draggable.</span>
  <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'rows'</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token variable">$draggable_row_found</span> <span class="token operator">=</span> <span class="token constant">FALSE</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-foreach">foreach</span> <span class="token punctuation">(</span><span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'rows'</span><span class="token punctuation">]</span> <span class="token keyword keyword-as">as</span> <span class="token operator">&amp;</span><span class="token variable">$row</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token comment" spellcheck="true">/** @var \Drupal\Core\Template\Attribute $row['attributes'] */</span>
      <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token function">empty</span><span class="token punctuation">(</span><span class="token variable">$row</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> <span class="token variable">$row</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span><span class="token function">hasClass</span><span class="token punctuation">(</span><span class="token string">'draggable'</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword keyword-if">if</span> <span class="token punctuation">(</span><span class="token operator">!</span><span class="token variable">$draggable_row_found</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
          <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'attributes'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'class'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'draggable-table'</span><span class="token punctuation">;</span>
          <span class="token variable">$draggable_row_found</span> <span class="token operator">=</span> <span class="token constant">TRUE</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>

  <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'#attached'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'library'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'olivero/olivero.table'</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span></code></pre><pre class="codeblock language-php"><code class=" language-php"><span class="token comment" spellcheck="true">/**
 * Implements hook_preprocess_HOOK() for views-view-table templates.
 */</span>
<span class="token keyword keyword-function">function</span> <span class="token function">olivero_preprocess_views_view_table</span><span class="token punctuation">(</span><span class="token operator">&amp;</span><span class="token variable">$variables</span><span class="token punctuation">)</span><span class="token punctuation">:</span> void <span class="token punctuation">{</span>
  <span class="token variable">$variables</span><span class="token punctuation">[</span><span class="token string">'#attached'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'library'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token punctuation">]</span> <span class="token operator">=</span> <span class="token string">'olivero/olivero.table'</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span></code></pre><p>
This change reduces unnecessary CSS on pages that do not contain HTML tables.</p>

---

