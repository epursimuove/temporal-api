<script setup lang="ts">
import { Temporal } from "@js-temporal/polyfill";
import { computed, ref, watchEffect } from "vue";
import type { ComputedRef, Ref } from "vue";
import LabelValueFormatter from "@/components/LabelValueFormatter.vue";

const optionsForLocaleString: Intl.DateTimeFormatOptions = {
  weekday: "long",
  year: "numeric",
  month: "long",
  day: "numeric",
  hour: "numeric",
  minute: "numeric",
};

const optionsForString: Temporal.ZonedDateTimeToStringOptions = {
  timeZoneName: "never",
};

const UNDEFINED: string = " ";

const locale: string = "en-SE";

type TimeZoneToUse = "utc" | "browser";

const timeZoneToBeUsed: Ref<TimeZoneToUse> = ref("utc");

const displayInZulu: ComputedRef<boolean> = computed(() => timeZoneToBeUsed.value === "utc");

// const browserTimeZone: string = new Intl.DateTimeFormat().resolvedOptions().timeZone;
const browserTimeZone: string = Temporal.Now.timeZoneId();
// const browserTimeZone: string = Temporal.TimeZone.from("Australia/Sydney") + "";

const instantNow: Ref<Temporal.Instant> = ref(Temporal.Now.instant().round("second"));

setInterval(() => (instantNow.value = Temporal.Now.instant().round("second")), 200);

const browserZonedDateTimeNow: ComputedRef<Temporal.ZonedDateTime> = computed(() =>
  instantNow.value.toZonedDateTimeISO(browserTimeZone),
);

const zuluZonedDateTimeNow: ComputedRef<Temporal.ZonedDateTime> = computed(() =>
  instantNow.value.toZonedDateTimeISO(Temporal.TimeZone.from("UTC")),
);

const rawInputValue: Ref<string> = ref(instantNow.value.toString());

const derivedInstant: Ref<Temporal.Instant | null> = ref(instantNow.value);

const browserZonedDateTimeDerivedInstant: ComputedRef<Temporal.ZonedDateTime | undefined> =
  computed(() => derivedInstant.value?.toZonedDateTimeISO(browserTimeZone));

const zuluZonedDateTimeDerivedInstant: ComputedRef<Temporal.ZonedDateTime | undefined> = computed(
  () => derivedInstant.value?.toZonedDateTimeISO(Temporal.TimeZone.from("UTC")),
);

const inputOkay: Ref<boolean> = ref(true);

const timestampRegExp = /^(\d\d\d\d-\d\d-\d\d)T(\d\d:\d\d:\d\d(\.\d{1,3})?)(Z|([+-]\d\d:\d\d))$/;

const epochSecondsRegExp = /^\d{1,10}$/;

const epochMilliSecondsRegExp = /^\d{11,13}$/;

watchEffect(() => {
  if (timestampRegExp.test(rawInputValue.value)) {
    derivedInstant.value = Temporal.Instant.from(rawInputValue.value);
    inputOkay.value = true;
  } else if (epochMilliSecondsRegExp.test(rawInputValue.value)) {
    derivedInstant.value = Temporal.Instant.fromEpochMilliseconds(parseInt(rawInputValue.value));
    inputOkay.value = true;
  } else if (epochSecondsRegExp.test(rawInputValue.value)) {
    derivedInstant.value = Temporal.Instant.fromEpochSeconds(parseInt(rawInputValue.value));
    inputOkay.value = true;
  } else {
    inputOkay.value = false;
    derivedInstant.value = null;
  }
});

const differenceFromNow: ComputedRef<Temporal.Duration | undefined> = computed(() => {
  const diff: Temporal.Duration | undefined = derivedInstant.value?.since(instantNow.value, {
    largestUnit: "hour",
  });

  return diff;
});

type HumanUnit = "years" | "months" | "days" | "hours" | "minutes";

const humanize: (diff: number, unit: HumanUnit) => string = (diff: number, unit: HumanUnit) => {
  const absDiff = Math.abs(diff);
  const prettifiedUnit = absDiff <= 1 ? unit.slice(0, -1) : unit;

  return diff > 0
    ? `In more than ${absDiff} ${prettifiedUnit}`
    : `More than ${absDiff} ${prettifiedUnit} ago`;
};

const examineHumanUnit: (unit: HumanUnit) => number | undefined = (unit: HumanUnit) => {
  const total = differenceFromNow.value?.total({
    unit: unit,
    relativeTo: zuluZonedDateTimeNow.value,
  });
  return total;
};

const differenceFromNowHuman: ComputedRef<string | undefined> = computed(() => {
  let result = undefined;

  if (differenceFromNow.value) {
    const units: HumanUnit[] = ["years", "months", "days", "hours", "minutes"];

    for (const unitsKey of units) {
      const value = examineHumanUnit(unitsKey);
      if (value && (value >= 1 || value <= -1)) {
        result = humanize(Math.trunc(value), unitsKey);
        break;
      }
    }
  }

  return result;
});
</script>

<template>
  <h1>Timestamp magician</h1>

  <label-value-formatter label="Browser default time zone" :value="browserTimeZone" />

  <div>
    <input type="radio" id="displayInZulu" value="utc" v-model="timeZoneToBeUsed" />

    <label for="displayInZulu"> Use <strong>Zulu</strong> time zone </label>
  </div>

  <div>
    <input type="radio" id="displayInBrowserTimeZone" value="browser" v-model="timeZoneToBeUsed" />

    <label for="displayInBrowserTimeZone">
      Use <strong>{{ browserTimeZone }}</strong> time zone
    </label>
  </div>

  <div class="your-input">
    <label for="rawInputValue">Timestamp</label>

    <input
      type="text"
      id="rawInputValue"
      size="29"
      maxlength="29"
      autofocus
      v-model="rawInputValue"
    />

    <span v-if="inputOkay" class="indicator">✔️</span>
    <span v-else class="indicator">❗️</span>
  </div>

  <label-value-formatter
    v-if="displayInZulu"
    label="Current time (Zulu)"
    :value="instantNow.toString()"
    :additional-value="zuluZonedDateTimeNow.toLocaleString(locale, optionsForLocaleString)"
  />

  <label-value-formatter
    v-else
    :label="`Current time (${browserTimeZone})`"
    :value="browserZonedDateTimeNow.toString(optionsForString)"
    :additional-value="browserZonedDateTimeNow.toLocaleString(locale, optionsForLocaleString)"
  />

  <label-value-formatter
    v-if="displayInZulu"
    label="Your defined timestamp (Zulu)"
    :value="derivedInstant?.toString() || UNDEFINED"
    :additional-value="
      zuluZonedDateTimeDerivedInstant?.toLocaleString(locale, optionsForLocaleString) || UNDEFINED
    "
  />

  <label-value-formatter
    v-else
    :label="`Your defined timestamp (${browserTimeZone})`"
    :value="browserZonedDateTimeDerivedInstant?.toString(optionsForString) || UNDEFINED"
    :additional-value="
      browserZonedDateTimeDerivedInstant?.toLocaleString(locale, optionsForLocaleString) ||
      UNDEFINED
    "
  />

  <label-value-formatter
    v-if="displayInZulu"
    label="Current day of week (Zulu)"
    :value="zuluZonedDateTimeNow.toLocaleString(locale, { weekday: 'long' })"
  />

  <label-value-formatter
    v-else
    :label="`Current day of week (${browserTimeZone})`"
    :value="browserZonedDateTimeNow.toLocaleString(locale, { weekday: 'long' })"
  />

  <label-value-formatter
    v-if="displayInZulu"
    label="Current week (Zulu)"
    :value="zuluZonedDateTimeNow.weekOfYear"
  />

  <label-value-formatter
    v-else
    :label="`Current week (${browserTimeZone})`"
    :value="browserZonedDateTimeNow.weekOfYear"
  />

  <label-value-formatter
    label="Difference/duration from now"
    :value="
      differenceFromNow
        ?.round({ largestUnit: 'year', relativeTo: zuluZonedDateTimeNow.toString() })
        .toString() || UNDEFINED
    "
    :additional-value="differenceFromNowHuman || UNDEFINED"
  />

  <label-value-formatter
    label="Difference in days"
    :value="differenceFromNow ? differenceFromNow.round('day').days : UNDEFINED"
  />

  <label-value-formatter
    :label="`Current time (epoch in seconds)`"
    :value="instantNow.epochSeconds"
  />

  <label-value-formatter
    :label="`Your defined timestamp (epoch in seconds and milliseconds)`"
    :value="derivedInstant?.epochSeconds || UNDEFINED"
    :additional-value="derivedInstant?.epochMilliseconds || UNDEFINED"
  />
</template>

<style scoped>
div {
  margin-block-end: 1rem;
}

.your-input {
  label {
    display: block;
  }
}

input {
  font-size: 1.5rem;
  font-family: var(--monospace-family);
}

@media screen and (max-width: 600px) {
  input {
    font-size: 1rem;
  }
}

.indicator {
  margin-left: 0.5rem;
}
</style>
