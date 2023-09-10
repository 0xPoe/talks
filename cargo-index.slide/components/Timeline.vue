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
        <text color="gray"><a href="https://www.github.com/kornelski">@kornelski</a> Submitted RFC</text>
      </a-timeline-item>
      <a-timeline-item>
        2020-11-24
        <text color="gray"><a href="https://www.github.com/jonhoo">@jonhoo</a> Submitted the first implementation <a href="https://www.github.com/rust-lang/Cargo/pull/8890">Cargo/PR#8890</a></text>
      </a-timeline-item>
      <a-timeline-item>
        2021-01-03
        <text color="gray"><a href="https://www.github.com/ehuss">@ehuss</a> Accepted the RFC</text>
      </a-timeline-item>
      <a-timeline-item>
        2022-03-10
        <text color="gray"><a href="https://www.github.com/arlosi">@arlosi</a> Submitted the final implementation <a  href="https://www.github.com/rust-lang/Cargo/pull/10470">Cargo/PR#10470</a> based on <a  href="https://www.github.com/rust-lang/Cargo/pull/8890">Cargo/PR#8890</a></text>
      </a-timeline-item>
      <a-timeline-item>
        2022-03-23
        <text color="gray"><a href="https://www.github.com/arlosi">@arlosi</a> Submitted the implementation in crates.io side <a  href="https://www.github.com/rust-lang/crates.io/pull/4661">crates.io/PR#4661</a> </text>
      </a-timeline-item>
      <a-timeline-item>
        2022-06-22
        <text color="gray"><a href="https://www.github.com/arlosi">@arlosi</a> Published <a href="https://blog.rust-lang.org/2022/06/22/sparse-registry-testing.html">Call for testing: Cargo sparse-registry</a> blog post</text>
      </a-timeline-item>
      <a-timeline-item>
        2023-01-20
        <text color="gray"><a href="https://www.github.com/arlosi">@arlosi</a> Stabilized Sparse Registry Feature in <a href="https://www.github.com/rust-lang/Cargo/pull/11224">Cargo/PR#11224</a></text>
      </a-timeline-item>
    </a-timeline>
</template>
