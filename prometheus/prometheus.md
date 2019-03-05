# Monitoring Your Elixir Application with Prometheus

![original](slide-bg.png)

Eric Oestrich
SmartLogic

^ Look through grapevine source

^ Simple local prometheus setup, excercise to viewer to stand up remote instance
  - Link to the ansible role we use

^ Show connecting a new client changes a dashboard

---

# SmartLogic

![original](slide-bg.png)

We build web and mobile applications

Have a project? We can help.
[smartlogic.io](https://smartlogic.io/) // [contact@smartlogic.io](mailto:contact@smartlogic.io)

---

![original](slide-bg.png)

- Context
- Setup
- Metrics
- Instrument
- Prometheus
- Grafana
- Alerting

---

# Prometheus

![original](slide-bg.png)

From metrics to insight

[prometheus.io](https://prometheus.io)

![inline](prometheus_logo.svg)

---

# Grafana

![original](slide-bg.png)

The open platform for beautiful analytics and monitoring

[grafana.com](https://grafana.com/)

![inline](grafana_logo.png)

---

# Other Options

![original](slide-bg.png)

- NewRelic
- CloudWatch
- DataDog
- Scout
- statsd
- Graphite
- ...

---

# Grapevine

![original](slide-bg.png)

Community site for text based games

[grapevine.haus](https://grapevine.haus)
[https://github.com/oestrich/grapevine](https://github.com/oestrich/grapevine)

![inline](grapevine.png)

^ Show a MUD through the grapevine web client, run around, generate lots of metrics, chat

---

# Setup

![original](slide-bg.png)

mix.exs

```elixir
[
  {:prometheus_ex, "~> 3.0"},
  {:prometheus_plugs, "~> 1.1"},
  {:telemetry, "~> 0.3"},
]
```

---

# Define Your Metrics Exporter

![original](slide-bg.png)

```elixir
defmodule Metrics.PlugExporter do
  use Prometheus.PlugExporter
end
```

---

# Add to Endpoint

![original](slide-bg.png)

[.code-highlight: 3]

```elixir
defmodule Web.Endpoint do
  # ...
  plug Metrics.PlugExporter
  # ...
end
```

---

# Metrics Setup

![original](slide-bg.png)

```elixir
defmodule Metrics.Setup do
  def setup do
    Metrics.PlugExporter.setup()
  end
end
```

---

# Add Setup to Your Application

![original](slide-bg.png)

[.code-highlight: 4]

```elixir
defmodule Grapevine.Application do
  def start(_type, _args) do
    # ...
    Metrics.Setup.setup()
    # ...

    Supervisor.start_link(children, opts)
  end
end
```

---

# Metrics!

![original](slide-bg.png)

Available at
[http://localhost:4100/metrics](http://localhost:4100/metrics)

```
# TYPE erlang_vm_atom_count gauge
# HELP erlang_vm_atom_count The number of atom ...
erlang_vm_atom_count 38398
# TYPE erlang_vm_atom_limit gauge
# HELP erlang_vm_atom_limit The maximum number ...
erlang_vm_atom_limit 1048576

# Many more
```

---

## Custom Instrumentation

![original](slide-bg.png)

---

# Our first Instrumenter

![original](slide-bg.png)

^ lib/metrics/account_instrumenter.ex

^[.code-highlight: 5-8]

```elixir
defmodule Metrics.AccountInstrumenter do
  use Prometheus.Metric

  def setup() do
    events = [
      [:create],
      [:session, :login],
      [:session, :logout],
    ]

    Enum.each(events, &setup_event/1)
  end
end
```

---

# Our first Instrumenter

![original](slide-bg.png)

^ lib/metrics/account_instrumenter.ex

^[.code-highlight: 5-8]

```elixir
defmodule Metrics.AccountInstrumenter do
  defp setup_event(event) do
    name = Enum.join(event, "_")
    name = "grapevine_accounts_#{name}"

    Counter.declare(
      name: String.to_atom("#{name}_total"),
      help: "Total of account event #{name}"
    )

    #...
  end
end
```

---

# Prometheus Naming

![original](slide-bg.png)

- Prefix with application name
- Use base units (seconds not milliseconds)
- Suffix with unit

[Prometheus Docs](https://prometheus.io/docs/practices/naming/)

---

# Add to our Setup module

![original](slide-bg.png)

[.code-highlight: 2]

```elixir
def setup do
  Metrics.AccountInstrumenter.setup()
  Metrics.PlugExporter.setup()
end
```

---

![original](slide-bg.png)

# Metrics

```
# TYPE grapevine_accounts_session_login_total counter
# HELP grapevine_accounts_session_login_total Total count ...
grapevine_accounts_session_login_total 0
```

---

# Triggering Events

![original](slide-bg.png)

0.3.0

```elixir
:telemetry.execute([:grapevine, :accounts, :create], 1)
```

---

# Triggering Events

![original](slide-bg.png)

0.4.0

```elixir
:telemetry.execute([:grapevine, :accounts, :create],
  %{value: 1})
```

---

# Register Telemetry Callbacks

![original](slide-bg.png)

^ Show lib/metrics/account_instrumenter.ex

[.code-highlight: 11-13]

```elixir
defmodule Metrics.AccountInstrumenter do
  defp setup_event(event) do
    name = Enum.join(event, "_")
    name = "grapevine_accounts_#{name}"

    Counter.declare(
      name: String.to_atom("#{name}_total"),
      help: "Total of account event #{name}"
    )

    # [:grapevine, :accounts, :create]
    event = [:grapevine, :accounts | event]
    :telemetry.attach(name, event, &handle_event/4, nil)
  end
end
```

---

## attach vs attach_many

![original](slide-bg.png)

---

# Handling Events

![original](slide-bg.png)

```elixir
def handle_event(event, count, metadata, config)

def handle_event([:grapevine, :accounts, :create], _, _, _) do
  Logger.info("New account created!")
  Counter.inc(name: :grapevine_accounts_create_total)
end
```

---

# Concurrency

![original](slide-bg.png)

- `:telemetry.execute` runs in the process that calls it
- Prometheus counters/gauges/etc are tracked via public ETS tables

---

# Viewing our Metrics

![original](slide-bg.png)

- `/metrics`
- Prometheus server
- Grafana

---

# Basic Prometheus Setup

![original](slide-bg.png)

- Scrape configs
- Rules

---

# Scrape Config

![original](slide-bg.png)

```yaml
scrape_configs:
  - job_name: 'grapevine'
    static_configs:
    - targets: ['localhost:4100']
```

---

![original](slide-bg.png)

![inline](prometheus-targets.png)

---

![original](slide-bg.png)

![inline](prometheus-graph.png)

---

# Rules

![original](slide-bg.png)

```yaml
alert: New User Registered
expr: increase(grapevine_account_create_total[5m]) > 0
annotations:
  subject: "Grapevine - New user registered"
  description: ""
```

---

![original](slide-bg.png)

![inline](prometheus-alerts.png)

---

# Connecting Grafana

![original](slide-bg.png)

^ Exercise for the viewer

---

# Dashboards

![original](slide-bg.png)

^ Show a dashboard

---

![original](slide-bg.png)

![inline](grafana-dashboard.png)

---

# Demo

![original](slide-bg.png)

^ https://prometheus.io/docs/practices/naming/

^ Show flow from registering a new user - see that alert
Opening a web client, seeing that alert
Step through the code that makes that ^ possible

^ Telemetry disconnects your metrics on any failure of the handle_event function

^ Show more of the telemetry setup

---

# Twitch

![original](slide-bg.png)

Live dev streams every Monday at 12PM Eastern

[twitch.tv/smartlogictv](https://twitch.tv/smartlogictv)

---

# Smart Software

![original](slide-bg.png)

[podcast.smartlogic.io](https://podcast.smartlogic.io)

![inline](smart-software.jpg)

---

# Questions?

![original](slide-bg.png)

[twitter.com/ericoestrich](https://twitter.com/ericoestrich)
[github.com/oestrich](https://github.com/oestrich)
[smartlogic.io](https://smartlogic.io/)

![inline](odo.png)
