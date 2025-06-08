---
title: Datadog Integration
sidebar_label: Datadog
hide_title: true
---

import ReactPlayer from 'react-player'

![Datadog](/img/integrations/datadog/datadog-logo.svg)

You can integrate Flagsmith with Datadog in two ways:

- [1. Integrate Flagsmith into your Datadog Dashboard](#1-integrate-flagsmith-into-your-datadog-dashboard)
- [2. Send Flag Change events to Datadog](#2-send-flag-change-events-to-datadog)

## 1. Integrate Flagsmith into your Datadog Dashboard

This integration lets you add a Flagsmith widget into your Datadog Dashboard so you can view and manage your flags
without having to leave the Datadog application.

![Datadog Dashboard Widget](/img/integrations/datadog/datadog-dashboard-widget.png)

The video below will walk you through the steps of adding the integration:

<ReactPlayer
    playing
    controls
    width="100%"
    height="460px"
    url='https://getleda.wistia.com/medias/76558s9yj7' />

## 2. Send Flag Change events to Datadog

The second type of integration allows you to send Flag change events in Flagsmith into your Datadog event stream.

![Datadog](/img/integrations/datadog/datadog-3.png)

1. Log into Datadog and go to Organization Settings > Access > API
2. Generate a new API key
3. Add the Datadog API key into Flagsmith (Integrations > Add Datadog Integration)
4. Add the Datadog URL into Flagsmith as the Base URL - (This is either `https://api.datadoghq.com/` or
   `https://api.datadoghq.eu/`)

![Datadog](/img/integrations/datadog/datadog-1.png)

![Datadog](/img/integrations/datadog/datadog-2.png)

Flag change events will now be sent to Datadog.
