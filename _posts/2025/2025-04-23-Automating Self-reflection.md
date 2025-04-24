---
title: "Automating Self-reflection"
layout: post
categories: [Home Automation, Self-reflection]
image:
  path: /assets/2025/reflection/reflection.png
  alt: Self-reflection
---

> _“We do not learn from experience… we learn from reflecting on experience.”_  
> John Dewey, _How We Think_ (1933)


Although my focus is usually on technical training,  I occasionally have workshops or trainings on executive skills. These workshops vary from planning, time-management to self-reflection. 

During one of these trainings, I received a handout with 30 “intentions” and matching “reflections.” The idea was simple: start the day with one of the intentions to "give direction to your day". And ending the day with the associated reflection to "enhance self-awareness, personal growth and find  strength". 

I liked the concept, especially the idea of finishing the workday with a brief reflection, leaving the work energy behind.

## Automating the process
Simply printing the list and planning to read it daily wasn’t realistic. I needed something automatic and persistent, a system that would guide me without extra steps.

So I turned to Home Assistant to send a random intention every morning and send the corresponding reflection in the evening.

## Daily intentions
I created an automation in Home Assistant that sends a random intention each morning via mobile notification:
```yaml
- alias: "Intention"
  description: Send daily intention in the morning.
  triggers:
    - trigger: time
      at: 07:30
  actions:
    - delay: "{ { (range(120, 720) | random | int) }}"
    - variables:
        place: 0
    - action: notify.mobile_app
      data:
        message: >-
          { % set intentions = [ / * list of intentions * / ] % }
          { % set intention_of_today = intentions | random % }
          Today's intention is: { { intention_of_today }}
        title: Daily intention
```

This worked great, and while debugging I received a randomly chosen intention. However, I also wanted the associated reflection at the end of the day. And this reflection wasn't random.

### Index
The reflections list used the same layout as the intention list, so the reflection associated with the intention was in the same list index.

Finding the index was easy.
```yaml
{ % set index = intentions.index(intention_of_today) % }
```
However, this index needed to be re-used in another automation.

After multiple test I found that storing the index in an `input_number` helper in the first automation worked best. For this, I used `variables` inside the automation, and moved the random intention from the message to a variable:

```yaml
  actions:
    - delay: "{ { (range(120, 720)|random|int) }}"
    - variables:
        intentions: >
          { {  [ / * list of intentions * / ] }}
        intention_of_today: "{ { intentions | random }}"
        place: "{ { intentions.index(intention_of_today) }}"
    - action: notify.mobile_app
      data:
        message: >-
           Today's intention is: { { intention_of_today }}
        title: Daily intention
    - action: input_number.set_value
      data_template:
        entity_id: input_number.daily_intention_number
        value: "{ { place | float }}"
```

## Daily self-reflection
To close off the day I created a second automation that will send the reflection based on the stored index. In this automation I also use `variables` to store the different values. And based on the previously defined index the appropriate reflection will be picked and send.

```yaml
  alias: "Reflection"
  description: Send daily reflection to phone in the afternoon.
  triggers:
    - trigger: time
      at: "16:30"
  actions:
    - delay: "{ { (range(120, 720)|random|int) }}"
    - variables:
        place: "{ { states('input_number.daily_intention_number') | int(0) }}"
        reflections: >
          { { [ / * list of reflections * / ] }}
        reflection_of_today: "{ { reflections[place] }}"
    - action: notify.mobile_app
      data:
        message: >-
          Today's reflection: { { reflection_of_today }}
        title: Daily reflection
```

## Visual feedback
To make it more tangible, I displayed the intention on a Home Assistant dashboard, with a  background image of the Scheveningen beach:

```yaml
- action: input_text.set_value
  data_template:
	entity_id: input_text.daily_intention
	value: "{ { intention_of_today }}"
```


![intention](/assets/2025/reflection/intention.png)
_Fig.1 Intention on dashboard_

## Reflection
This small automation has made the daily intention/reflection exercise easier to be reminded. And with this helps me to briefly reflect to close the work day.