---
layout: post
title:  "Custom Events in Home Assistant"
author: skalavala
categories: [ homeassistant, event ]
image: assets/images/12.jpg
featured: true
hidden: true
---

Looking for a quick and easy way to fire custom events and subscribe to them in Home Assistant? Check out the following code...

## Home Assistant Custom Events

<p>The following two automations shows you how you can raise an event with [additional] data and catch that event and the corresponding data.</p>

```yaml
automation:
  - alias: Fire Event
    trigger:
      platform: state
      entity_id: switch.kitchen
    action:
      event: my_test_event
      event_data:
        foo: "bar"

  - alias: Capture Event
    trigger:
      platform: event
      event_type: my_test_event
    action:
      - service: script.notify_me
        data_template:
          message: "Test Event Captured with data foo: {% raw %}{{ trigger.event.data.foo }}{% endraw %}"
```
