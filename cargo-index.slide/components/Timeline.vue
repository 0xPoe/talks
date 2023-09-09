<script>
  import { Timeline, TimelineItem } from 'ant-design-vue';

  export default {
    components: {
      ATimeline: Timeline,
      ATimelineItem: TimelineItem,
    },
  };

</script>

<template>
    <a-timeline>
      <a-timeline-item>
        2019-10-18
        <text color="gray"><a>@kornelski</a> Submitted RFC</text>
      </a-timeline-item>
      <a-timeline-item>
        2020-11-24
        <text color="gray"><a>@jonhoo</a> Submitted the first implementation <a>Cargo/PR#8890</a></text>
      </a-timeline-item>
      <a-timeline-item>
        2021-01-03
        <text color="gray"><a>@ehuss</a> Accepted the RFC</text>
      </a-timeline-item>
      <a-timeline-item>
        2022-03-10
        <text color="gray"><a>@arlosi</a> Submitted the final implementation <a>Cargo/PR#10470</a> based on <a>Cargo/PR#8890</a></text>
      </a-timeline-item>
      <a-timeline-item>
        2022-03-23
        <text color="gray"><a>@arlosi</a> Submitted the implementation in crates.io side <a>crates.io/PR#4661</a> </text>
      </a-timeline-item>
      <a-timeline-item>
        2022-06-22
        <text color="gray"><a>@arlosi</a> Published <a>Call for testing: Cargo sparse-registry</a> blog post</text>
      </a-timeline-item>
      <a-timeline-item>
        2023-01-20
        <text color="gray"><a>@arlosi</a> Stabilized Sparse Registry Feature in <a>Cargo/PR#11224</a></text>
      </a-timeline-item>
    </a-timeline>
</template>
