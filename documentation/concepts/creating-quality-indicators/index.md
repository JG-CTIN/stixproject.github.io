---
layout: flat
title: 5 Tips to Create Quality Indicators
---

The STIX team has seen indicators from many different sources and has learned a lot about what makes a good STIX indicator. Here's some ideas on how to make sure you're creating the best indicators possible.

## 1. Don't just translate observables to indicators

The best way to create low-value indicators is to take a bunch of things that you saw and directly encode them into indicators. As an example: you get an e-mail with a malicious payload and want to share it as an indicator. A naive approach is to just [encode that e-mail into a STIX indicator](/documentation/idioms/malicious-email-attachment) and send it on its way with the original `Subject`, the full `Body`, `To` address, and `From` address.

See the problem? By including all of that data in the indicator what you're saying is that the indicator is only a hit if all of those fields match, greatly limiting how usable it is for others. Instead of that approach, identify which fields are actually indicative of the malicious activity and include only those in the indicator. If you have other data that you feel adds context or might be useful you can include it as an observable in a [Sighting](/data-model/{{site.current_version}}/indicator/SightingType).

Some fields in CybOX objects should almost never be used in indicators because they're either ephemeral, specific to a certain environment, or might be interpreted or calculated differently by consumers of the indicator. For example, the process ID (PID) field on the Process Object changes from execution to execution and the Peak_Entropy field on the File Object may be calculated differently by different tools. For some suggestions on which fields are often useful, take a look at the [list the MAEC project maintains](http://maec-to-stix.readthedocs.org/en/latest/indicator_extraction/granular_config_defaults.html) to create indicators from malware samples.

## 2. Don't be afraid to put benign data in indicators (if you know what you're doing)

Note that following that first tip doesn't mean that you should never put "benign" data in your indicators: sometimes specific combinations of data points that are individually benign can indicate malicious activity. For example, a DNS query to Google is not unusual. A DNS query to Google for a random-looking domain is less common but still benign (many content distribution networks use randomized subdomain names). Combined with the presence of a particular registry key, running process, or other network traffic however it can be the calling card of some sort of malware.

If you go down this route you should make sure to use composition (either indicator or observable) to ensure that the indicator doesn't trigger solely on benign data. You shouldn't have a standalone indicator that looks for a DNS query to Google, but having it as part of an AND with other targeted observables can be useful.

## 3. Think about false positives and false negatives

In statistics, this is called balancing your [type one errors and type two errors](https://en.wikipedia.org/wiki/Type_I_and_type_II_errors). Basically, no indicator you create is going to perfectly detect the adversary: it will either detect things that aren't actually malicious (false positive), or it will not detect things that are malicious (false negative), or, in most cases, it will do both. You can and should try to minimize these as much as possible but to some extent it's a balancing act: creating more specific indicators will minimize false positives at the expense of false negatives, while creating less specific indicators will minimize false negatives at the expense of false positives.

[Doug Wilson](https://twitter.com/dallendoug) from FireEye has given some very good [talks](https://www.first.org/conference/2015/program#pvalidating-and-improving-threat-intelligence-indicators) on determining the ideal balance by understanding what you want the indicators to be used for: when creating indicators for detection you should probably bias in favor of minimizing false positives while when creating indicators for incident response you should probably bias in the opposite direction and create "noisy" indicators.

## 3. Include context

If you're just representing a list of IP addresses or malicious domains that you know to be bad without any additional information then STIX is probably not for you. It will create a lot of overhead and not allow you to express anything new. On the other hand, threat analysts understand that blacklists without context are not all that useful: if you get a hit, what do you do? What problem do you have? [Cisco](http://blogs.cisco.com/security/moving-from-indicators-of-compromise-to-actionable-content-fast) has a blog post where they explain how they use this additional context to make indicators truly actionable.

STIX gives you the ability to express rich context about indicators, whether it's the [confidence](/data-model/{{site.current_version}}/stixCommon/ConfidenceType) that the indicator is actually malicious, the [associated malware](/documentation/idioms/malware-hash), [attribution](/documentation/idioms/indicators-to-campaigns) to a particular campaign, or potential [courses of action](/data-model/{{site.current_version}}/coa/CourseOfActionType). Leverage these capabilities if you have that data and give your consumers the best chance to put your data to good use.

## 4. Consider patterning

Another capability that STIX (well, CybOX) gives you is the ability to include patterning on fields in your indicators. You don't have to match an exactly e-mail subject, but can rather match just a portion of it. Similarly, an IP address might match a subnet range or even just 5 consecutive addresses. One caution: many consumers are not capable of understanding patterning, so use with caution (see tip #5).

PS: One manifestation of this capability leads to the most common mistake that we see in STIX indicators: forgetting to include a `@condition` attribute on your indicator observables. Even if it's just "Equals", you need to use this field so that consumers understand that the observables are patterns of things they might see and not something that you're saying that you actually saw.

## 5. Understand your consumers

To sometimes contradicts #4, but as a producer you should understand the capabilities that your consumers have to ingest your indicators. Can they support patterning? Can they support the richer network connection object or simply the IP address object? Can they support CybOX at all or is it better to use Snort or YARA?

When creating indicators for mass consumption this can be tricky, but as a general rule think about what 80% of consumers can handle and target that. Currently, our estimation is that producers creating content for mass consumption should limit themselves to the more basic objects and conditions as represented in the [simple indicator sharing profile](/language/profiles/samples/stix_{{site.current_version}}_sample_indicator_sharing_profile_r1.xlsx).

One thing you can do to support _all_ levels of consumers is create multiple indicators and use indicator composition to create an OR condition. For example, if you have an SSDEEP hash and a SHA1 hash, rather than including them in the same indicator instead split them into separate indicators and OR them together. That way consumers that understand SSDEEP can match the fuzzy hash and potentially detect more while consumers that only understand SHA1 are still able to get value from that portion (and can ignore the SSDEEP hash).

## Bonus tip: not everything is about indicators

Not everything you share needs to be an indicator! Have a malware analysis that you want to share? Encode it in [MAEC](https://maec.mitre.org) and share it via a STIX [TTP](/data-model/{{site.current_version}}/ttp/TTPType). Have a new attack pattern that you've seen but not any indicators for how to find it? Share it as a STIX TTP w/ CAPEC reference. New attribution for an attack? STIX [Threat Actor](/data-model/{{site.current_version}}/ta/ThreatActorType) or [Campaign](/data-model/{{site.current_version}}/campaign/CampaignType).

Those can all be later related to indicators but if you don't have the indicator right away there's no reason to force it.
