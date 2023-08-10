# Franklin RUM Conversion tracking extension

Adds conversion tracking functionality to Helix RUM Collection (client-side)

## Installation

```bash
git subtree add --squash --prefix plugins/rum-conversion git@github.com:adobe/franklin-rum-conversion.git main
```

You can then later update it from the source again via:
```bash
git subtree pull --squash --prefix plugins/rum-conversion git@github.com:adobe/franklin-rum-conversion.git main
```

## Initialization
In your `script.js` find the method `loadLazy()`.
At the end of the method add the following code:

```
  const context = {
    getMetadata,
    toClassName,
  };
  const { initConversionTracking } = await import('../plugins/rum-conversion/src/index.js');
  initConversionTracking.call(context, document, '');
```
Please, note that `getMetadata` and `toClassName` methods should be imported from `lib-franklin.js` in your `script.js`

## Usage

At the moment, the conversion tracking that is used to report conversions to RUM is both too broad and too narrow.

* Too broad: any `click` on the page will be counted as a conversion, not just clicks on relevant elements such as the "sign-up", "free demo", "price quote" CTAs.
* Too narrow: if the user navigates away from the current page, and converts later, this can still indicate a successful conversion, just one that has been delayed

With this extension, developers can declare arbitrary elements to be conversion targets that track a conversion when they are `clicked` (or `submitted` in case of forms). Each conversion can carry either a conversion name such as "requested quote" or a conversion value such as a dollar amount (e.g. the value of the shopping cart upon checkout). To do so, the `sampleRUM.convert` function is used.

The conversion names and conversion values can later on be used in reporting the effectiveness of an experiment.
### Practitioner defined conversions
_**Identifying the user actions to track**_

In order to setup conversions a practitioner must define a metadata property called `Conversion Element` which can have the values: `< Link | Labeled Link | Form >`

* `Link`:  Clicks on any link `<a href="...">` will be tracked as conversions.
* `Form`: form submissions in the page will be tracked as conversions.
* `Labeled Link`: Only links specified in the metadata property `Conversion Link Labels` will be considered for tracking conversions.

The three values can be combined, although if `Link` is configured, `Labeled Link` would be redundant.

In case of `Conversion Element = Labeled Link`, to define the list of links for which we want to track clicks as conversions we use the metadata property:

* `Conversion Link Labels`:  Comma separated list of link labels that will be tracked as conversions. The link label is the inner text of the link.

![conversion-element](https://user-images.githubusercontent.com/43381734/218769859-8302c97d-98ad-4bfc-b7c4-0edcc0aa0f08.png)

_**Conversion Names for Link clicks**_

Practitioners can assign a conversion name to each of the link clicks. A metadata property for each link will be defined:

* `Conversion Name (<Link Label>)` : Link label as explained above is the inner text of the link. The value of this property will be used as conversion name when a user clicks the link.
* `Conversion Name`: it is also possible to use a default conversion name for all links in the document.

_By default_ If no conversion name is defined for a link, the link inner text converted `toClassName` will be used as conversion name. That is the inner text to lower case, replacing white spaces by dashes.

![link-conversion-metadata](https://user-images.githubusercontent.com/43381734/218726528-83570d0c-d2d6-4a00-a70d-46bcab15669d.png)

_**Conversion Names and Values for Form submission**_

While conversion names for link clicks are defined exclusively in the document metadata, the conversion name for a form submission can be defined by adding the property `Conversion Name`, either in the **section** metadata where the form resides (could be in a fragment document), or in the main document metadata.

* `Conversion Name`: the value of the property will be used as conversion name to track the form submission.

_By default_ If no conversion name is defined for a form, if the form is included as part of a fragment document, the path of the fragment `toClassName()` is used. Last fallback is the form id.

Practitioners can also define a **conversion value** for form submissions. Conversion value should be a numeric value, and is normally related to the monetary aspect of the conversion. The conversion value is defined with another section metadata property called `Conversion Value Field`, allowed values for this property are:

* Id of the form field whose value we want to use as conversion value
* Name of the form field whose value we want to use as conversion value
* Label of the field whose value we want to use as conversion value

![form-conversion-metadata](https://user-images.githubusercontent.com/43381734/218726040-81fb4d04-9a91-495e-a23b-50fcafd86a75.png)

### Developer defined conversions
For more specific requirements it is also possible for developers to invoke the conversion API using the following method:

`sampleRUM.convert(cevent, cvalueThunk, element, listenTo = []) `

`cevent` is the conversion name `cvalueThunk` can be the conversion value or a function that calculates the conversion value `element` is the element that generates the conversion `listenTo` is the array of events we want to listen to to generate a conversion.

This method has 2 modes:

* listener registration mode: If the method is called with `element` and `listenTo` values it will register a listener on the element for the given events, every time the event is triggered a conversion with the given arguments will be tracked.
* conversion tracking mode: If the method is called with empty `listenTo` it will track a conversion using as conversion name the `cevent` and/or `cvalueThunk` as conversion value.

### Integration with Adobe Data Layer
After any conversion is registered in the RUM the conversion is also pushed to the Adobe Data Layer so it can be tracked by other products such as Adobe Analytics.
