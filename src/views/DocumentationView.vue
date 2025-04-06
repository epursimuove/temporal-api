<template>
  <h1>Documentation</h1>

  <p>
    Define a timestamp, either an epoch value (in milliseconds or seconds) or ISO 8601 timestamp up
    to milliseconds precision.
  </p>
  
  <p>
    If you define one timestamp, it is <em>compared against current time</em> (and the difference auto-updates). If you define two timestamps, they are <em>compared against each other</em>.
  </p>

  <p>Examples of the ISO 8601 timestamp format:</p>

  <ul>
    <li>
      <code>2024-05-04T17:59:34Z</code>
    </li>
    <li>
      <code>2024-05-04T17:59:34-10:00</code>
    </li>
    <li>
      <code>2024-05-04T17:59:34+03:00</code>
    </li>
    <li>
      <code>2024-05-04T17:59:34.123Z</code>
    </li>
  </ul>

  <p>Examples of epoch values in seconds:</p>

  <ul>
    <li>
      <code>1743953123</code>
    </li>
    <li>
      <code>1653953100</code>
    </li>
  </ul>

  <p>Examples of epoch values in milliseconds:</p>

  <ul>
    <li>
      <code>1743953123000</code>
    </li>
    <li>
      <code>1653953100000</code>
    </li>
  </ul>

  <p>
    You can display the zoned datetime values in either Zulu/UTC time zone (default) or the time
    zone configured for your browser.
  </p>
</template>
<script setup lang="ts">
</script>
