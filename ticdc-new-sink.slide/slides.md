---
theme: seriph
class: 'text-center'
highlighter: shiki
lineNumbers: false
info: |
  TiCDC Sink Component.

  Learn more at [TiFlow](https://github.com/pingcap/tiflow/tree/v6.5.1/cdc/sinkv2)
drawings:
  presenterOnly: true
css: unocss
---

# TiCDC Sink Component

A Deep Dive

Based on TiCDC [v6.5.1](https://github.com/pingcap/tiflow/tree/v6.5.1)

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Begin <carbon:arrow-right class="inline"/>
  </span>
</div>

---
transition: slide-up
---

# Rustin Liu

<div class="leading-8 opacity-80">
PingCAPer.<br/>
Data Replication Team.<br/>
Cargo Contributor.<br/>
Rustup Maintainer.<br/>
</div>

<div my-10 grid="~ cols-[40px_1fr] gap-y4" items-center justify-center>
  <div i-ri-github-line op50 ma text-xl/>
  <div><a href="https://github.com/hi-rustin" target="_blank">hi-rustin</a></div>
  <div i-ri-twitter-line op50 ma text-xl/>
  <div><a href="https://twitter.com/hi_rustin" target="_blank">hi_rustin</a></div>
  <div i-ri-firefox-line op50 ma text-xl/>
  <div><a href="https://hi-rustin.rs" target="_blank">hi-rustin.rs</a></div>
</div>

<img src="https://avatars.githubusercontent.com/u/29879298?v=4" rounded-full w-40 abs-tr mt-22 mr-22/>

<div flex="~ gap2">
</div>

---
transition: slide-up
layout: center
---

<div text-6xl fw100>
  Agenda
</div>

<br>

<div class="grid grid-cols-[3fr_2fr] gap-4">
  <div class="border-l border-gray-400 border-opacity-25 !all:leading-12 !all:list-none my-auto">

  - Architecture
  - Old & New Sink Module Design
  - Table Sink Deep Dive
  - MySQL Sink Deep Dive
  - Q&A

  </div>
</div>

---
transition: slide-up
layout: center
---

# Architecture

---
transition: slide-up
layout: center
---

<div class="arch">
<div>

# Architecture

</div>

<div
  class="relation"
>

- A TiCDC cluster has only one owner.
- A capture will have multiple processors.
- A processor can only process one changefeed.
- A changefeed can synchronize multiple tables.

</div>

<div>

```plantuml {scale: 0.8}
@startuml
!theme plain
package "TiKV" {
  gRPC - [TiKV CDC]
}

node "Owner" {
  [Scheduler]
  [DDLSink]
  package "DDL" as DDL1 {
    [OwnerSchemaStorage]
    [OwnerDDLPuller] --> [gRPC]
  }
}

node "Processor" {
  [ProcessorMounter]
  [ProcessorSink] #FF6655
  package "DDL" as DDL2 {
    [ProcessorSchemaStorage]
    [ProcessorDDLPuller] --> [gRPC]
  }
  package "Changefeed1" {
    package "Table1 Pipeline" {
      [Puller1] --> [gRPC]
      [Sorter1]
      [TableSink1] #Yellow
    }
    package "Table2 Pipeline" {
      [Puller2] --> [gRPC]
      [Sorter2]
      [TableSink2] #Yellow
    }
  }
}

database "MySQL/Kafka" {
  [MySQL]
  [Broker]
}

[DDLSink] --> [MySQL]
[DDLSink] --> [Broker]
DDL1 --> [DDLSink]
[OwnerSchemaStorage] ..> [OwnerDDLPuller] : use
[ProcessorSink] --> [MySQL]
[ProcessorSink] --> [Broker]
[Sorter1] ..> [ProcessorMounter] : use
[Sorter2] ..> [ProcessorMounter] : use
[TableSink1] ..> [ProcessorSink] : use
[TableSink2] ..> [ProcessorSink] : use
[ProcessorMounter] ..> DDL2 : use
[Puller1] --> [Sorter1]
[Sorter1] --> [TableSink1]
[Puller2] --> [Sorter2]
[Sorter2] --> [TableSink2]
@enduml
```

</div>
</div>

<style>
.arch {
  display: flex;
}

.arch img {
  margin-top: -80px;
}

.relation {
  position: absolute;
  z-index: 1;
  left: 120px;
  top: 60px;
  font-size: 12px;
}

h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 50%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  writing-mode: vertical-rl;
  text-orientation: mixed;
}
</style>

---

# Table Pipeline

Each changefeed creates a processor, and each processor maintains multiple table pipelines.

### Pipeline
<br>
<br>

```mermaid {scale: 1.5}
flowchart LR
    puller((PullerNode)) --> sorter((SorterNode)) --> mounter((Mounter)) --> sink((SinkNode))
```

---
transition: slide-up
layout: center
---

# Old Sink Design

---

<div class="relation">

<div class="title">

# Old Sink Design

</div>
<div class="uml">

```plantuml {scale: 1}
@startuml
!theme plain
package "TiKV" {
  gRPC - [TiKV CDC]
}

node "Processor" {
  [ProcessorMounter]
  package "Changefeed1" {
    package "Table1 Pipeline" {
      [Puller1] --> [gRPC]
      [Sorter1]
      [TableSink1] #Yellow
    }
    package "Table2 Pipeline" {
      [Puller2] --> [gRPC]
      [Sorter2]
      [TableSink2] #Yellow
    }
    package "Sink Manager" {
      folder "Combination" as Combination {
        [MySQLSink] #FF6655
        [BufferSink] #FF6655
      }
    }
  }
}

database "MySQL"

note right of [MySQLSink]
  It can be either
  MQ Sink or BlackHoleSink.
end note

[MySQLSink] --> [MySQL]
[Sorter1] ..> [ProcessorMounter] : use
[Sorter2] ..> [ProcessorMounter] : use
[TableSink1] ..> Combination : use
[TableSink2] ..> Combination : use
Combination .[#FF6655].> [TableSink1] : use
Combination .[#FF6655].> [TableSink2] : use
[Puller1] --> [Sorter1]
[Sorter1] --> [TableSink1]
[Puller2] --> [Sorter2]
[Sorter2] --> [TableSink2]
@enduml
```

</div>
</div>

<style>
.relation {
  display: flex;
  justify-content: flex-start;
}

.relation img {
  height: 500px;
}

.relation .title {
  flex-grow: 4;
}

.relation .uml {
  flex-grow: 2;
}

h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 50%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  writing-mode: vertical-rl;
  text-orientation: mixed;
}
</style>

---
transition: slide-up
---

<div class="arch">
<div>

# Old Sink Module Abstract

</div>

<div>
  <br/>
  <br/>
  <br/>

```plantuml
@startuml
!theme plain
folder "Sink Manager" as SM {
  class BS as "Buffer Sink" #FF6655
  folder "MySQL Sink" as MS {
    class DML as "DML Sink" #FF6655
    class DDL as "DDL Sink" #FF6655
  }
}
class SN as "Sink Node" #Yellow
class TS as "Table Sink" #Yellow

SN *-- TS : use
SM *-[#FF6655]- TS : manage
TS *-[#FF6655]- SM : use

note left of TS
  Table Sink Managed by Sink Manager
  and Table Sink also use Sink Manager.
  ⚠️This is a circular dependency.
end note

note right of MS
  MySQL Sink is a combination of
  DML Sink and DDL Sink.
  ⚠️However, only one of these is used per module.
end note
@enduml
```

</div>
</div>

<style>
.arch {
  display: flex;
}

.arch img {
  margin-top: -80px;
}

h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 50%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  writing-mode: vertical-rl;
  text-orientation: mixed;
}
</style>

---

# Old Data Sequence (Sync)

## Row Change Data Sequence

```plantuml {scale: 0.9}
@startuml
!theme plain
SinkNode <[bold,#FF6655]- SinkNode: ⚠️added to buffer
SinkNode -> TableSink: buffer is full and SinkNode calls EmitRowChangedEvents
SinkNode <-[bold,#FF6655]- TableSink: ⚠️added to buffer
SinkNode -> TableSink: calls FlushRowChangedEvents
TableSink -> "ProcessorSink(SinkManager)": calls EmitRowChangedEvents
TableSink <-[bold,#FF6655]- "ProcessorSink(SinkManager)": ⚠️added to buffer
loop BufferSink
  "ProcessorSink(SinkManager)" -> "ProcessorSink(SinkManager)": BufferSink buffer is full
  "ProcessorSink(SinkManager)" -> Worker: calls EmitRowChangedEvents of MySQLSink
end
Worker -> MySQL: Execute SQL
Worker <- MySQL: SQL Result


note left of SinkNode #FF6655
  Buffer One.
end note

note left of TableSink #FF6655
  Buffer Two.
end note

note left of "ProcessorSink(SinkManager)" #FF6655
  Buffer Three.
end note

@enduml
```

---
transition: slide-up
layout: two-cols
---

# Old Sink Module Abstract

```go
type Sink interface {
	// FIXME: some sink implementation is not thread-safety, but they should be.
	EmitRowChangedEvents(ctx context.Context, rows ...*model.RowChangedEvent) error

	// TryEmitRowChangedEvents is thread-safety and non-blocking.
	TryEmitRowChangedEvents(ctx context.Context, rows ...*model.RowChangedEvent) (bool, error)

	// FIXME: some sink implementation is not thread-safety, but they should be.
	EmitDDLEvent(ctx context.Context, ddl *model.DDLEvent) error

	// FlushRowChangedEvents flushes each row which of commitTs less than or
	// equal to `resolvedTs` into downstream.
	// FIXME: some sink implementation is not thread-safety, but they should be.
	FlushRowChangedEvents(ctx context.Context, tableID model.TableID, resolvedTs uint64) (uint64, error)

	// FIXME: some sink implementation is not thread-safety, but they should be.
	EmitCheckpointTs(ctx context.Context, ts uint64, tables []model.TableName) error
	...
}
```

::right::

<br/>
<br/>

```plantuml
@startuml
!theme plain
interface S as "Sink"
class SM as "Sink Manager"
class TS as "Table Sink" #Yellow
package "Combination" #DDDDDD {
  class BS as "Buffer Sink"
  class MQS as "MQ Sink" #FF6655
  class MS as "MySQL Sink" #FF6655
}
SM *-- TS : manage
TS *-- SM  : use
SM *-- BS : use
S <|.. TS : implement
S <|.. MQS : implement
S <|.. MS : implement
S <|.. BS : implement

@enduml
```

---
transition: slide-up
---

# Old Sink Problems

- Circular dependency
- Too many buffers
- Leak table information everywhere
- Call stack is too deep and all calls are synchronous
- Abstraction is very complicated and too many functions in the interface
- Some implementations are not thread-safe, but they should be

---
transition: slide-up
layout: center
---

# New Sink Design

---

<div class="relation">

<div class="title">

# New Sink Design

</div>
<div class="uml">

```plantuml {scale: 1}
@startuml
!theme plain
package "TiKV" {
  gRPC - [TiKV CDC]
}

node "Processor" {
  [ProcessorMounter]
  package "Changefeed1" {
    package "Table1 Pipeline" {
      [Puller1] --> [gRPC]
      [Sorter1]
      [TableSink1] #Yellow
    }
    package "Table2 Pipeline" {
      [Puller2] --> [gRPC]
      [Sorter2]
      [TableSink2] #Yellow
    }
    package "Event Sink" {
      [Worker1] #FF6655
      [Worker2] #FF6655
    }
  }
}

database "MySQL"

note right of [Event Sink]
  It can be
  MySQL Sink、MQ Sink、Cloud Storage Sink or BlackHoleSink.
end note

[Worker1] --> [MySQL]
[Worker2] --> [MySQL]
[Sorter1] ..> [ProcessorMounter] : use
[Sorter2] ..> [ProcessorMounter] : use
[TableSink1] ..> [Event Sink] : use
[TableSink2] ..> [Event Sink] : use
[Puller1] --> [Sorter1]
[Sorter1] --> [TableSink1]
[Puller2] --> [Sorter2]
[Sorter2] --> [TableSink2]
@enduml
```

</div>
</div>

<style>
.relation {
  display: flex;
  justify-content: flex-start;
}

.relation img {
  height: 500px;
}

.relation .title {
  flex-grow: 4;
}

.relation .uml {
  flex-grow: 2;
}

h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 50%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  writing-mode: vertical-rl;
  text-orientation: mixed;
}
</style>

---
transition: slide-up
---

# New Sink Module Abstract

<br/>
<br/>

```plantuml
@startuml
!theme plain
class TS as "Table Sink" #Yellow
class ES as "Event Sink" #FF6655
class MQS as "MQ Event Sink"
class TXNS as "Transaction Event Sink"
class CSS as "Cloud Storage Event Sink"

TS *-- ES : use

ES <|.. MQS : implement
ES <|.. TXNS : implement
ES <|.. CSS : implement

class DDLS as "DDL Sink"
class DDLMQS as "MQ DDL Sink"
class DDLTXNS as "Transaction DDL Sink"
class DDLCSS as "Cloud Storage DDL Sink"

DDLS <|.. DDLMQS : implement
DDLS <|.. DDLTXNS : implement
DDLS <|.. DDLCSS : implement
@enduml
```

<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Table Sink](https://github.com/pingcap/tiflow/tree/v6.5.1/cdc/sinkv2/tablesink)
· [Event Sink](https://github.com/pingcap/tiflow/blob/v6.5.1/cdc/sinkv2/eventsink/event_sink.go) · [DDL Sink](https://github.com/pingcap/tiflow/tree/v6.5.1/cdc/sinkv2/ddlsink) · [MQ Event Sink](https://github.com/pingcap/tiflow/tree/v6.5.1/cdc/sinkv2/eventsink/mq) · [Transaction Event Sink](https://github.com/pingcap/tiflow/tree/v6.5.1/cdc/sinkv2/eventsink/txn) · [Cloud Storage Event Sink](https://github.com/pingcap/tiflow/tree/v6.5.1/cdc/sinkv2/eventsink/cloudstorage)

---
transition: slide-up
---

# New Data Sequence (Async)

## Row Change Data Sequence

<br/>

```plantuml
@startuml
!theme plain
participant TS1 as "Table Sink1"
participant TS2 as "Table Sink2"
participant TXNS as "Transaction Event Sink"
participant MW as "MySQL Worker"
participant M as "MySQL Server"

TS1 -> TXNS: call WriteEvents
TS2 -> TXNS: call WriteEvents
TXNS -[bold,#FF6655]> MW: Dispatch Txn Events(Conflict detection)
MW -> M: Execute SQL
M --> MW: Execute SQL Result
MW --> TS1: call Callback
MW --> TS2: call Callback

note left of TS1 #FF6655
  Only one buffer.
end note

@enduml
```
---
transition: slide-up
---

# Table Sink

## Abstract

```go{all|7|11|15}
// TableSink is the interface for table sink.
// It is used to sink data in table units.
type TableSink interface {
	// AppendRowChangedEvents appends row changed events to the table sink.
	// Usually, it is used to cache the row changed events into table sink.
	// This is a not thread-safe method. Please do not call it concurrently.
	AppendRowChangedEvents(rows ...*model.RowChangedEvent)
	// UpdateResolvedTs writes the buffered row changed events to the eventTableSink.
	// Note: This is an asynchronous and not thread-safe method.
	// Please do not call it concurrently.
	UpdateResolvedTs(resolvedTs model.ResolvedTs) error
	// GetCheckpointTs returns the current checkpoint ts of table sink.
	// For example, calculating the current progress from the statistics of the table sink.
	// This is a thread-safe method.
	GetCheckpointTs() model.ResolvedTs
	// Close closes the table sink.
	// We should make sure this method is cancellable.
	Close(ctx context.Context)
}
```

---
transition: slide-up
---

# Table Sink

## Implementation


```go{all|2|9|4|6|5}
// EventTableSink is a table sink that can write events.
type EventTableSink[E eventsink.TableEvent] struct {
	...
	maxResolvedTs   model.ResolvedTs
	backendSink     eventsink.EventSink[E]
	progressTracker *progressTracker
	...
	// NOTICE: It is ordered by commitTs.
	eventBuffer []E
	state       state.TableSinkState
	...
}
```

```go{0|all}
type EventSink[E TableEvent] interface {
	// WriteEvents writes events to the sink.
	// This is an asynchronously and thread-safe method.
	WriteEvents(events ...*CallbackableEvent[E]) error
	// Close closes the sink.
	Close() error
}
```

---
transition: slide-up
---

<div class="arch">
<div>

# Row Change Data Sequence

</div>

<div>
<br/>
<br/>

```plantuml {scale: 0.9}
@startuml
!theme plain
participant SN as "Sink Node"
participant TS as "Table Sink"
participant PT as "Progress Tracker"
participant TXNS as "Transaction Event Sink"
participant MW as "MySQL Worker"
participant M as "MySQL Server"

group #lightblue Add Event
SN -> TS: call AppendRowChangedEvents
SN <- TS: added to buffer
end

group #lightyellow Update ResolvedTs
SN -> TS: call UpdateResolvedTs
TS -> PT: call AddEvent
TS <- PT: return a callback
TS -> PT: call AddResolvedTs
TS <- PT: added to records
TS -> TXNS: call WriteEvents with callback
TS <- TXNS: added to Conflicts Detector
SN <- TS: updated resolvedTs
end
group #lightgreen Async Write Events
TXNS -> MW: Dispatch Txn Events(Conflict detection)
MW -> M: Execute SQL
M -> MW: Execute SQL Result
MW --> PT: call Callback
end

group #pink Get CheckpointTs
SN -> TS: call GetCheckpointTs
TS -> PT: call advance
TS <- PT: return checkpointTs
end
@enduml
```

</div>
</div>

<style>
.arch {
  display: flex;
}

.arch img {
  margin-top: -80px;
}

.relation {
  position: absolute;
  z-index: 1;
  left: 120px;
  top: 60px;
  font-size: 12px;
}

h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 50%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  writing-mode: vertical-rl;
  text-orientation: mixed;
}
</style>

---
transition: slide-up
---

# Table Sink

## Implementation

```go {all|7|17|23|24} {maxHeight:'80%'}
// UpdateResolvedTs advances the resolved ts of the table sink.
func (e *EventTableSink[E]) UpdateResolvedTs(resolvedTs model.ResolvedTs) error {
	...
	e.maxResolvedTs = resolvedTs

	i := sort.Search(len(e.eventBuffer), func(i int) bool {
		return e.eventBuffer[i].GetCommitTs() > resolvedTs.Ts
	})
	...
	resolvedEvents := e.eventBuffer[:i]
	...
	e.eventBuffer = append(make([]E, 0, len(e.eventBuffer[i:])), e.eventBuffer[i:]...)
	resolvedCallbackableEvents := make([]*eventsink.CallbackableEvent[E], 0, len(resolvedEvents))
	for _, ev := range resolvedEvents {
		ce := &eventsink.CallbackableEvent[E]{
			Event:     ev,
			Callback:  e.progressTracker.addEvent(),
			SinkState: &e.state,
		}
		resolvedCallbackableEvents = append(resolvedCallbackableEvents, ce)
	}
	// Do not forget to add the resolvedTs to progressTracker.
	e.progressTracker.addResolvedTs(resolvedTs)
	return e.backendSink.WriteEvents(resolvedCallbackableEvents...)
}
```

---
transition: slide-up
---

# Table Sink

## Progress Tracker

<br/>

### Can we use a simple counter to track the progress?

<br/>

<v-click>

### What about the performance?

</v-click>
<br/>

<v-click>

### How can we make it faster?

</v-click>

<br/>
<v-click>

##### BitMap

```go{all|1|2}
// 0000000000000000000000000000000000000000000000000000000000000000 ->
// 0000000000000000000000000000000000000000000000000000000000001000
```

</v-click>

---
transition: slide-up
---

# Table Sink

## Progress Tracker

<br/>

```go{all|6|7-10|11-14}
type progressTracker struct {
	// Following fields are protected by `mu`.
	mu sync.Mutex
	...
	// Used to generate the next eventID.
	nextEventID uint64
	// Every received event is a bit in `pendingEvents`.
	pendingEvents [][]uint64
	// When old events are flushed the buffer should be released.
	nextToReleasePos uint64
	// The resolvedTs of the table sink.
	resolvedTsCache []pendingResolvedTs
	// The position that the next event which should be check in `advance`.
	nextToResolvePos uint64
	lastMinResolvedTs model.ResolvedTs
}
```

---
transition: slide-up
---

# Table Sink

## Progress Tracker


```go {all|3-5|7-13|14-18|21-26} {maxHeight:'80%'}
func (r *progressTracker) addEvent() (postEventFlush func()) {
	...
	eventID := r.nextEventID
	bit := eventID % 64
	r.nextEventID += 1

	bufferCount := len(r.pendingEvents)
	if bufferCount == 0 || (uint64(len(r.pendingEvents[bufferCount-1])) == r.bufferSize && bit == 0) {
		// If there is no buffer or the last one is full, we need to allocate a new one.
		buffer := make([]uint64, 0, r.bufferSize)
		r.pendingEvents = append(r.pendingEvents, buffer)
		bufferCount += 1
	}

	if bit == 0 {
		// If bit is 0 it means we need to append a new uint64 word for the event.
		r.pendingEvents[bufferCount-1] = append(r.pendingEvents[bufferCount-1], 0)
	}
	lastBuffer := r.pendingEvents[bufferCount-1]

	// Set the corresponding bit to 1.
	// For example, if the eventID is 3, the bit is 3 % 64 = 3.
	// 0000000000000000000000000000000000000000000000000000000000000000 ->
	// 0000000000000000000000000000000000000000000000000000000000001000
	// When we advance the progress, we can try to find the first 0 bit to indicate the progress.
	postEventFlush = func() { atomic.AddUint64(&lastBuffer[len(lastBuffer)-1], 1<<bit) }
	return
```

---
transition: slide-up
---

# Table Sink

## Progress Tracker

```go {all} {maxHeight:'80%'}
func (r *progressTracker) addResolvedTs(resolvedTs model.ResolvedTs) {
	...
	// If there is no event or all events are flushed, we can update the resolved ts directly.
	if r.nextEventID == 0 || r.nextToResolvePos >= r.nextEventID {
		// Update the checkpoint ts.
		r.lastMinResolvedTs = resolvedTs
		return
	}
	// Sometimes, if there are no events for a long time and a lot of resolved ts are received,
	// we can update the last resolved ts directly.
	tsCacheLen := len(r.resolvedTsCache)
	if tsCacheLen > 0 {
		// The offset of the last resolved ts is the last event ID.
		// It means no event is adding. We can update the resolved ts directly.
		if r.resolvedTsCache[tsCacheLen-1].offset+1 == r.nextEventID {
			r.resolvedTsCache[tsCacheLen-1].resolvedTs = resolvedTs
			return
		}
	}
	r.resolvedTsCache = append(r.resolvedTsCache, pendingResolvedTs{
		offset:     r.nextEventID - 1,
		resolvedTs: resolvedTs,
	})
}
```

---
transition: slide-up
---

# Table Sink

## Progress Tracker

```go {all|2-10|13-32|34-48|49-59} {maxHeight:'80%'}
func (r *progressTracker) advance() model.ResolvedTs {
	// `pendingEvents` is like a 3-dimo bit array. To access a given bit in the array,
	// use `pendingEvents[idx1][idx2][idx3]`.
	// The first index is used to access the buffer.
	// The second index is used to access the uint64 in the buffer.
	// The third index is used to access the bit in the uint64.
	offset := r.nextToResolvePos - r.nextToReleasePos
	idx1 := offset / (r.bufferSize * 64)
	idx2 := offset % (r.bufferSize * 64) / 64
	idx3 := offset % (r.bufferSize * 64) % 64
	for {
		...
		currBitMap := atomic.LoadUint64(&r.pendingEvents[idx1][idx2])
		if currBitMap == math.MaxUint64 {
			// Move to the next uint64 word (maybe in the next buffer).
			idx2 += 1
			if idx2 >= r.bufferSize {
				idx2 = 0
				idx1 += 1
			}
			r.nextToResolvePos += 64 - idx3
			idx3 = 0
		} else {
			// Try to find the first 0 bit in the word.
			for i := idx3; i < 64; i++ {
				if currBitMap&uint64(1<<i) == 0 {
					r.nextToResolvePos += i - idx3
					break
				}
			}
			break
		}
	}
	// Try to advance resolved timestamp based on `nextToResolvePos`.
	if r.nextToResolvePos > 0 {
		for len(r.resolvedTsCache) > 0 {
			cached := r.resolvedTsCache[0]
			if cached.offset <= r.nextToResolvePos-1 {
				...
				r.resolvedTsCache = r.resolvedTsCache[1:]
				if len(r.resolvedTsCache) == 0 {
					r.resolvedTsCache = nil
				}
			} else {
				break
			}
		}
	}
	// If a buffer is finished, release it.
	for r.nextToResolvePos-r.nextToReleasePos >= r.bufferSize*64 {
		r.nextToReleasePos += r.bufferSize * 64
		// Use zero value to release the memory.
		r.pendingEvents[0] = nil
		r.pendingEvents = r.pendingEvents[1:]
		if len(r.pendingEvents) == 0 {
			r.pendingEvents = nil
		}
	}
	return r.lastMinResolvedTs
}
```
---
transition: slide-up
layout: center
---

# MySQL Sink

---
transition: slide-up
---

# MySQL Sink Data Sequence (Async)

## Row Change Data Sequence

<br/>

```plantuml
@startuml
!theme plain
participant TS1 as "Table Sink1"
participant TS2 as "Table Sink2"
participant TXNS as "Transaction Event Sink"
participant MW as "MySQL Worker"
participant M as "MySQL Server"

TS1 -> TXNS: call WriteEvents
TS2 -> TXNS: call WriteEvents
TXNS -[bold,#FF6655]> MW: Dispatch Txn Events(Conflict detection)
MW -> M: Execute SQL
M --> MW: Execute SQL Result
MW --> TS1: call Callback
MW --> TS2: call Callback

note left of TS1 #FF6655
  Only one buffer.
end note

@enduml
```

---
transition: slide-up
---

# MySQL Sink

## Conflict Detection - Union Set

```sql{all|1|2|3|4|5|6}
DML1: INSERT INTO t VALUES (1, 2);
DML2: INSERT INTO t VALUES (2, 3);
DML3: UPDATE t SET pk = 4, uk = 3 WHERE pk = 2;
DML4: DELETE FROM t WHERE pk = 1;
DML5: REPLACE INTO t VALUES (1, 3);
DML6: INSERT INTO t VALUES (5, 6);
```

```plantuml
@startuml
!theme plain

node "PK:1" as PK1 #pink
node "UK:2" as UK1 #pink
node "Group 1" as G1 #736060
node "PK:1" as PK4 #d4b4b4
node "UK:2" as UK4 #d4b4b4

PK1 --> G1
UK1 --> G1
PK4 -up-> G1
UK4 -up-> G1

node "PK:2" as PK2 #yellow
node "UK:3" as UK2 #yellow
node "PK:2(Old)" as PK32 #lightyellow
node "UK:3(Old)" as UK32 #lightyellow
node "PK:4(New)" as PK31 #lightyellow
node "UK:3(New)" as UK31 #lightyellow
node "Group 2" as G2 #736060
PK2 --> G2
UK2 --> G2
PK31 -up-> G2
UK31 -up-> G2
PK32 -up-> G2
UK32 -up-> G2

node "PK:1" as PK5 #red
node "UK:3" as UK5 #red

PK5 -right-> G1
UK5 -left-> G2

node "PK:5" as PK6 #green
node "UK:6" as UK6 #green
node "Group 3" as G3 #736060
PK6 --> G3
UK6 --> G3
@enduml
```

---
transition: slide-up
---

# MySQL Sink

## Conflict Detection - DAG

- Node: Transaction received by Conflict Detector, that has not been executed.
- Edge: T2 -> T1, T1 exists one edge to T2, only if T1 modifies the same key as T2.
<br/>

> We can ignore T2 -> T1, if there exists one path T2 -> Ta -> Tb ... -> Tx -> T1.


```sql{all|1|2|3|4|5|6}
DML1: INSERT INTO t VALUES (1, 2);
DML2: INSERT INTO t VALUES (2, 3);
DML3: UPDATE t SET pk = 4, uk = 3 WHERE pk = 2;
DML4: DELETE FROM t WHERE pk = 1;
DML5: REPLACE INTO t VALUES (1, 3);
DML6: INSERT INTO t VALUES (5, 6);
```

```plantuml
@startuml
!theme plain

node "T1(PK:1, UK:2)" as T1 #pink
node "T2(PK:2, UK:3)" as T2 #yellow
node "T3(old: PK:2, UK:3, new: PK:4, UK:3)" as T3 #lightyellow
node "T4(PK:1, UK:2)" as T4 #d4b4b4
node "T5(PK:1, UK:3)" as T5 #red

T3 -right-> T2
T4 -right-> T1
T5 -right-> T4
T5 --> T3

node "T6(PK:5, UK:6)" #green
@enduml
```

---
layout: center
---

# Q&A

<br/>
<br/>

## Do you have any questions?

---
layout: center
class: text-center
---

# Learn More

[Documentations](https://docs.pingcap.com/tidb/dev/ticdc-overview) · [GitHub](https://github.com/pingcap/tiflow)  · [How to write a new sink](https://hi-rustin.rs/TiCDC-Sink-%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97/)
