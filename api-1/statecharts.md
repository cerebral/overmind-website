# statecharts

Statecharts is a configuration factory, just like **merge**, **namespaced** and **lazy**. It allows you to restrict what actions are to be run in certain states.

## statechart

The factory function you use to wrap an Overmind configuration. You add one or multiple charts to the configuration, where the key is the _id_ of the chart.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {}> = {}

export default statechart(config, chart)
```

## initial

Define the initial state of the chart. When a parent chart enters a transition state, any nested chart will move to its initial transition state.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {
  STATE_A: void
}> = {
  initial: 'STATE_A'
}

export default statechart(config, chart)
```

## states

Defines the transition states of the chart. The chart can only be in one of these states at any point in time.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {
  STATE_A: void
  STATE_B: void
}> = {
  initial: 'STATE_A',
  states: {
    STATE_A: {},
    STATE_B: {}
  }
}

export default statechart(config, chart)
```

## entry

When a transition state is entered you can optionally run an action. It also runs if it is the initial state.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {
  STATE_A: void
  STATE_B: void
}> = {
  initial: 'STATE_A',
  states: {
    STATE_A: {
      entry: 'someActionName'
    },
    STATE_B: {}
  }
}

export default statechart(config, chart)
```

## exit

When a transition state is changed, any exit defined in current transition state will be run first. Nested charts in a transition state with an exit defined will run before parents.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {
  STATE_A: void
  STATE_B: void
}> = {
  initial: 'STATE_A',
  states: {
    STATE_A: {
      entry: 'someActionName',
      exit: 'someOtherActionName'
    },
    STATE_B: {}
  }
}

export default statechart(config, chart)
```

## on

Unlike traditional statecharts Overmind uses its actions as transition types. This keeps a cleaner chart definition and when using Typescript the actions will have correct typing related to their payload. The actions defined are the only actions allowed to run. They can optionally lead to a new transition state, even conditionally lead to a new transition state.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {
  STATE_A: void
  STATE_B: void
}> = {
  initial: 'STATE_A',
  states: {
    STATE_A: {
      on: {
        // Allow execution, but stay on this transition state
        someAction: null,

        // Move to new transition state when executed
        someOtherAction: 'STATE_B',

        // Conditionally move to a new transition state
        someConditionalAction: {
          target: 'STATE_B',
          condition: state => state.isTrue
        }
      }
    },
    STATE_B: {}
  }
}

export default statechart(config, chart)
```

## nested

A nested statechart will operate within its parent transition state. The means when the parent transition state is entered or exited any defined **entry** and **exit** actions will be run. When the parent enters its transition state the **initial** state of the child statechart\(s\) will be activated.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const nestedChart: Statechart<typeof config, {
  FOO: void
  BAR: void
}> = {
  initial: 'FOO',
  states: {
    FOO: {
      on: {
        transitionToBar: 'BAR'
      }
    },
    BAR: {
      on: {
        transitionToFoo: 'FOO'
      }
    }
  }
}

const chart: Statechart<typeof config, {
  STATE_A: typeof nestedChart
  STATE_B: void
}> = {
  initial: 'STATE_A',
  states: {
    STATE_A: {
      on: {
        transitionToStateB: 'STATE_B'
      },
      chart: nestedChart
    },
    STATE_B: {
      on: {
        transitionToStateA: 'STATE_A'
      }
    }
  }
}

export default statechart(config, chart)
```

## parallel

Multiple statecharts will run in parallel. Either for the factory configuration or nested charts. You can add the same chart multiple times behind different ids.

```typescript
import { Statechart, statechart } from 'overmind/config'
import * as actions from './actions'
import { state } from './state'

const config = {
  actions,
  state
}

const chart: Statechart<typeof config, {
  STATE_A: void
  STATE_B: void
}> = {
  initial: 'STATE_A',
  states: {
    STATE_A: {
      on: {
        transitionToStateB: 'STATE_B'
      }
    },
    STATE_B: {
      on: {
        transitionToStateA: 'STATE_A'
      }
    }
  }
}

export default statechart(config, {
  chartA: chart,
  chartB: chart
})
```

{% hint style="info" %}
Also the nested **chart** property of charts can contain parallel charts
{% endhint %}

## matches

The matches API is used in your components to identify what state your charts are in. It is accessed on the **state**.

```typescript
// Given that you have added statecharts to the root configuration
state.matches({
  STATE_A: true
})

// Nested chart
state.matches({
  STATE_A: {
    FOO: true
  }
})

// Parallel
state.matches({
  chartA: {
    STATE_A: true
  },
  chartB: {
    STATE_B: true
  }
})

// Negative check
state.matches({
  chartA: {
    STATE_A: true
  },
  chartB: {
    STATE_B: false
  }
})
```

