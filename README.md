# adapt-capture

### Introduction

Capture is a small toolkit to help template authors use component data and user input.

### Required knowledge

An understanding of JSON and Handlebars.

### Rationale

When there is a requirement for a new way to present data in Adapt the typical workflow is to create a new component. Code is written to obtain the data and render it via a template. Such components are often non-interactive and the purpose of the code may only be to move data. In these cases coding may be unnecessary and reducing the reliance on code would increase the potential size of the author base. The purpose of Capture therefore is to provide a means for authors to use only templating to produce new ways to present data.

Capture achieves this:
- with some simple functionality to facilitate templating
- by instruction through example and with good documentation

### Implementation

It is possible to access the data for any component using Capture. This data can be standard model data which is defined in the JSON during authoring or it can be user input that is provided at runtime.

Standard components that accept user input are the question components. These components are well-defined and so Capture provides some helpers to facilitate accessing and displaying user input for these components.

To work with custom components Capture provides a basic mechanism to make arbitrary user input available to templating: the author opens the relevant template(s), identifies which input elements are of interest and gives them an identifying tag. Capture can then record user input for these elements and make it available as part of the component data.

Capture is composed of two plugins: the `adapt-captureSummary` component provides example templates together with a selection of Handlebars helpers to facilitate working with the data. It also provides a simple method to open a printable template in a new window. The `adapt-capture` extension provides these templates with the data; it is responsible for capturing user input in components and recording it against the corresponding component models.

### Selecting components of interest

Components of interest are given a `_capture` configuration which determines whether the component data will be available to templating. To make a component available set the property `_isEnabled` to `true`:
```json
"_capture": {
    "_isEnabled": true
}
```
The component data is made available via the `components` list.

### Standard and non-standard components

Standard components have the prefix `adapt-contrib` in their name and consist of the familiar presentation and question types. Some of these components accept user input. In fact, those that do accept user input are all question components. There are some helpers and example partial templates to assist the author when working with standard question components.

Non-standard refers to any other component that might be found in an Adapt project. The design principle of Capture is to make working with these components easy. User input to these components can be captured by locating the template and tagging elements of interest. In the following example a custom component has a single input element to be captured. It is tagged by adding a `data-capture` attribute:

```handlebars
<input type="text" data-capture="default">
```

If a component has multiple elements to be captured then each element is given a different value for the attribute:

```handlebars
<input type="text" data-capture="input1">
<input type="text" data-capture="input2">
```

Once the elements are tagged in this way Capture will record any user input they receive. This data is stored in the component model on the `__captures` property. This property is an object with each key corresponding to the value of a `data-capture` attribute. The value associated with the key is the user input. From the above examples the user input would be accessed as follows:

```javascript
__captures.default

```
and
```javascript
__captures.input1
__captures.input2

```

### Accessing component data in a template

One way to access component data within a template is via the value of its `_id` property:

```handlebars
{{#with components.c-60}}
    <div class="title">{{{displayTitle}}}</div>
    <div class="userInput">{{{__captures.default}}}</div>
{{/with}}
```

Alternatively provide a `_captureId` in the configuration, e.g:

```json
"_capture": {
    "_isEnabled": true,
    "_captureId": "acmeComponent"
}
```
and use in the template:

```handlebars
{{#with components.acmeComponent}}
    <div class="title">{{{displayTitle}}}</div>
    <div class="userInput">{{{__captures.default}}}</div>
{{/with}}
```

All component data can be accessed in this way for captured components. To see what data is available type the following at the command prompt within your browser developer tools:

`require('coreJS/adapt').findById(modelId).attributes;`

where `modelId` is the value of the component `_id` property.

### Using helpers

Some component data can be accessed directly, such as the `displayTitle` seen previously. Other data can be accessed more easily with the use of a Handlebars helper.

The `adapt-captureSummary` component provides some helpers to make working with standard components easy. The following example shows how the `getMcqOptionText` helper can be used to list the options a user selected when an MCQ was answered:

```handlebars
{{#with components.c-60}}
    {{#if _isSubmitted}}
        <div><em>You answered:</em></div>
        <ul>
            {{#each _userAnswer}}
                {{#if this}}
                    <li><strong>{{{getMcqOptionText ../_id @index}}}</strong></li>
                {{/if}}
            {{/each}}
        </ul>
    {{else}}
        <div><em>This component has not yet been answered.</em></div>
    {{/if}}
{{/with}}
```
Note the presence of hard-coded strings in the above example. This can and should be avoided. See the section "Global configuration" for more information on how to avoid hard-coding strings in templates.

Several helpers exist to make working with standard components easier. See the section "Standard helpers" for more information.

### Partials

When there are multiple components it is often helpful to be able to reuse a fragment of markup in a template. Handlebars partials serve exactly this purpose. The fragment is moved into a separate template and then invoked when necessary.

The `adapt-captureSummary` component includes some example partials for the standard components that accept user input (graphical MCQ, matching, MCQ, Slider and Text input).

For custom components or any instance where a component should use a particular partial; this can be specified in the configuration:

```json
"_capture": {
    "_isEnabled": true,
    "_template": "customInput"
}
```
Combining these ideas an example template could be as follows:
```handlebars
{{#if _capture._template}}
    {{> (lookup _capture '_template') __captureConfig=../__captureConfig}}
{{else}}
    {{#equals _component "gmcq"}}
        {{> printMcq __captureConfig=../__captureConfig}}
    {{/equals}}
    {{#equals _component "mcq"}}
        {{> printMcq __captureConfig=../__captureConfig}}
    {{/equals}}
    {{#equals _component "matching"}}
        {{> printMatching __captureConfig=../__captureConfig}}
    {{/equals}}
    {{#equals _component "slider"}}
        {{> printSlider __captureConfig=../__captureConfig}}
    {{/equals}}
    {{#equals _component "textinput"}}
        {{> printTextInput __captureConfig=../__captureConfig}}
    {{/equals}}
{{/if}}
```
N.B. the example partials in this guide are prefixed `"print"` as they have been taken from a template that demonstrates reformatting data for print. Partials as equally useful for use in screen (i.e. page) templates.

### Iterating over multiple components

In previous examples explicit references to components were made in the templates (`c-60`, `acmeComponent`). When there are a large number of components to consider this may be impractical. One method to deal with this is to iterate over the components using a loop and partials:

```handlebars
{{#each components}}
    {{#if _capture._template}}
        {{> (lookup _capture '_template') __captureConfig=../__captureConfig}}
    {{else}}
        {{#equals _component "gmcq"}}
            {{> printMcq __captureConfig=../__captureConfig}}
        {{/equals}}
        {{#equals _component "mcq"}}
            {{> printMcq __captureConfig=../__captureConfig}}
        {{/equals}}
        {{#equals _component "matching"}}
            {{> printMatching __captureConfig=../__captureConfig}}
        {{/equals}}    
    {{/if}}
{{/each}}
```

### Grouping components

When iterating over the `components` list as in previous examples, the order of output is according to the order of the components in the list. Sometimes it may be desirable to control the order of output. While it is possible to access individual components directly, this requires explicit references to be made in the template, which could become tedious if there are a significant number of components. It is also cumbersome and difficult to maintain if component identifiers are changed. To address this, components to be captured can be grouped:
```json
"_capture": {
    "_isEnabled": true,
    "_group": "textinputExamples"
}
```
When templating each named group can be found in the `groups` list. Each group has a list of its associated components. For example:

```handlebars
{{#each groups.textinputExamples}}
    {{#if _capture._template}}
        {{> (lookup _capture '_template') __captureConfig=../__captureConfig}}
    {{else}}
        {{> printTextInput __captureConfig=../__captureConfig}}
    {{/if}}
{{/each}}
```

### Global configuration

A standard practice in Adapt is to locate strings outside of templates to maintain data separation and to facilitate maintenance and localisation. Capture uses a global configuration which can be obtained for use in a template by using the following statement:
```handlebars
{{import_captureConfig}}
```
To use this in partials the configuration is passed to the partial when it is invoked, for example:
```handlebars
{{> printMcq __captureConfig=../__captureConfig}}
```
Strings can then be accessed easily, as in the following example:
```handlebars
{{__captureConfig.strings.unanswered}}
```

### Standard helpers

A small number of Handlebars helpers have been defined for use with standard components.

`getMcqOptionText`

This helper takes the component `_id` and the index of the option and returns the option text, e.g:
```handlebars
{{getMcqOptionText "c-60" 0}}
```
`getMatchingItemText`

This helper takes the component `_id` and the index of the item and returns the item text, e.g:
```handlebars
{{getMatchingItemText "c-60" 0}}
```
`getMatchingOptionText`

This helper takes the component `_id`, the item index, the option index and returns the item text, e.g:
```handlebars
{{getMatchingOptionText "c-60" 0 0}}
```
Note that these helpers are not necessary to access the data. The previous `getMatchingOptionText` example could be rewritten as:
```handlebars
{{#with components.c-60}}
    {{#each _items.[0]._options}}
        {{#equals _index 0}}
            {{{text}}}
        {{/equals}}
    {{/each}}
{{/with}}
```
The following extract is from the printMatching partial:
```handlebars
<ul>
    {{#each _userAnswer}}
        <li>
            <span class="item-title">{{{getMatchingItemText ../_id @index}}}</span>
            <div class="option-title">
                <em>{{../__captureConfig.strings.youAnswered}}</em>
                <strong>{{{getMatchingOptionText ../_id @index this}}}</strong>
            </div>
        </li>
    {{/each}}
</ul>
```
Continuing the example of rewriting without use of the helper could yield:
```handlebars
<ul>
    {{#each _userAnswer}}
        {{#with (lookup ../_items @index)}}
            <span class="item-title">{{{text}}}</span>
            {{#each _options}}
                {{#equals _index ../..}}
                    <div class="option-title">
                        <em>{{../../../__captureConfig.strings.youAnswered}}</em>
                        <strong>{{{text}}}</strong>
                    </div>
                {{/equals}}
            {{/each}}
        {{/with}}
    {{/each}}
</ul>
```
### Additional data

Accessing component data and user input has been discussed up to this point together with the global Capture configuration. The Adapt `course` and `config` objects can be accessed with the following directives:
```handlebars
{{import_adaptCourse}}

{{import_adaptConfig}}
```
If only the course name is required then it is unnecessary to use the `import_adaptCourse` directive because a `courseName` property is available to templating.

Today's date is provided to templating and can be accessed via the (top level) property `dateFormatted`. The date format can be configured in the global Capture configuration. See also https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toLocaleString for more information.

Capture exposes various pieces of personal information via the properties `name`, `firstname`, `lastname`, and `id`. A formatted version of the `name` property is available via the `nameFormatted` property. This can be configured in the global Capture configuration.
