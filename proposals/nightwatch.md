# Nightwatch
version: alpha

## Motivation
Nightwatch is designed as a cloud native alert component based on kubernetes. It aims to watch important components and alert in bad status simply and easily.

## Goals
1. Declarative alert rules
2. Extendable alert adaptors
3. Based on all kubernetes resources (contains custom resource)
4. Lightweight, user friendly, no storage, no large memory

## Non-Goals
1. Alert for "current" status but not time series data of status

## Design
### Overview
Nightwatch is based on kubernetes resource and will be implemented by kubernetes CRD feature.
The main flow of alert is as below.

1. Watch resource defined in alert rule.
2. Start a clock when resource matches the rule.
3. If clock is timeout and resource does not change the state, alert will be sent.
4. If resource changes the state but matches the rule again in jitter duration, clock will not stop.

### Definition

```go
type Alert struct {
    TypeMeta `json:",inline"`
    ObjectMeta `json:"metadata,omitempty"`

    Spec AlertSpec `json:"spec"`
    Status AlertStatus `json:"status"`
}

type AlertSpec struct {
    // Reference defines kubernetes object reference
    Reference ObjectReference `json:"reference"`

    // Selector defines label selector to filter objects
    Selector *LabelSelector `json:"selector,omitempty"`

    // rules defines alert rules
    Rules []Rule `json:"rules"`
}

type AlertStatus struct {
    LastAlertTime time.Time `lastAlertTime,omitempty`
    NextAlertTime time.Time `nextAlertTime,omitempty`
}

type Rule struct {
    // Path defines path of object.
    Path string `json:"path"`

    // Expression defines expression to compare expected and actual.
    Expression Expression `json:"expression"`

    // Timeout defines waiting time after the resource
    // matches the rule.
    // If the resource doesn't recover from the status,
    // alert will be sent.
    Timeout time.Duration `json:"timeout"`

    // Jitter defines minimum waiting time to stop
    // alert clock if the resource recovers from the
    // status defined in this rule.
    // It is used to avoid status jitter because the
    // resource is always trying to recover automatically
    Jitter time.Duration `json:"jitter"`

    // MaxJitterTimes defines maximum times that status jitter
    // happens
    MaxJitterTimes int `json:"maxJitterTimes"`
}
```

### Why say no to time series data
Time series data analysis will cost too much resource and make whole alert system more unstable.
If it is really needed, `Prometheus` is a good choice for these alert rules.

