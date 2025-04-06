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

const compareAgainstNow: Ref<boolean> = ref(true);

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
  instantNow.value.toZonedDateTimeISO("UTC"),
);

const rawInputValue: Ref<string> = ref(instantNow.value.toString());

const rawAdditionalInputValue: Ref<string> = ref(instantNow.value.add(Temporal.Duration.from({hours: 20})).toString());

const derivedInstant: Ref<Temporal.Instant | null> = ref(instantNow.value);

const additionalDerivedInstant: Ref<Temporal.Instant | null> = ref(instantNow.value);

const browserZonedDateTimeDerivedInstant: ComputedRef<Temporal.ZonedDateTime | undefined> =
  computed(() => derivedInstant.value?.toZonedDateTimeISO(browserTimeZone));

const zuluZonedDateTimeDerivedInstant: ComputedRef<Temporal.ZonedDateTime | undefined> = computed(
  () => derivedInstant.value?.toZonedDateTimeISO("UTC"),
);

const browserZonedDateTimeAdditionalDerivedInstant: ComputedRef<Temporal.ZonedDateTime | undefined> =
  computed(() => additionalDerivedInstant.value?.toZonedDateTimeISO(browserTimeZone));

const zuluZonedDateTimeAdditionalDerivedInstant: ComputedRef<Temporal.ZonedDateTime | undefined> = computed(
  () => additionalDerivedInstant.value?.toZonedDateTimeISO("UTC"),
);

const inputOkay: Ref<boolean> = ref(true);

const additionalInputOkay: Ref<boolean> = ref(true);

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
    derivedInstant.value = Temporal.Instant.fromEpochMilliseconds(
      parseInt(rawInputValue.value) * 1000,
    );
    inputOkay.value = true;
  } else {
    inputOkay.value = false;
    derivedInstant.value = null;
  }
});

watchEffect(() => {
  if (timestampRegExp.test(rawAdditionalInputValue.value)) {
    additionalDerivedInstant.value = Temporal.Instant.from(rawAdditionalInputValue.value);
    additionalInputOkay.value = true;
  } else if (epochMilliSecondsRegExp.test(rawAdditionalInputValue.value)) {
    additionalDerivedInstant.value = Temporal.Instant.fromEpochMilliseconds(parseInt(rawAdditionalInputValue.value));
    additionalInputOkay.value = true;
  } else if (epochSecondsRegExp.test(rawAdditionalInputValue.value)) {
    additionalDerivedInstant.value = Temporal.Instant.fromEpochMilliseconds(
      parseInt(rawAdditionalInputValue.value) * 1000,
    );
    additionalInputOkay.value = true;
  } else {
    additionalInputOkay.value = false;
    additionalDerivedInstant.value = null;
  }
});

const differenceFromNow: ComputedRef<Temporal.Duration | undefined> = computed(() => {
  const diff: Temporal.Duration | undefined = derivedInstant.value?.since(instantNow.value, {
    largestUnit: "hour",
  });

  return diff;
});

const differenceBetweenTimestamps: ComputedRef<Temporal.Duration | undefined> = computed(() => {
  let diff: Temporal.Duration | undefined;

  if (additionalDerivedInstant.value && derivedInstant.value) {
    diff = derivedInstant.value.since(additionalDerivedInstant.value, {
      largestUnit: "hour",
    });
  }

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

const humanizeTwoTimestamps: (diff: number, unit: HumanUnit, exactDiff: number) => string = (diff: number, unit: HumanUnit, exactDiff: number) => {
  const absDiff = Math.abs(diff);
  const prettifiedUnit = absDiff <= 1 ? unit.slice(0, -1) : unit;

  return diff === exactDiff
    ? `Exactly ${absDiff} ${prettifiedUnit}`
    : `More than ${absDiff} ${prettifiedUnit}`;
};

const examineHumanUnit: (unit: HumanUnit) => number | undefined = (unit: HumanUnit) => {
  const total = differenceFromNow.value?.total({
    unit: unit,
    relativeTo: zuluZonedDateTimeNow.value,
  });
  return total;
};

const examineHumanUnitBetweenTimestamps: (unit: HumanUnit) => number | undefined = (unit: HumanUnit) => {
  const total = differenceBetweenTimestamps.value?.total({
    unit: unit,
    relativeTo: zuluZonedDateTimeAdditionalDerivedInstant.value,
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

const differenceBetweenTimestampsHuman: ComputedRef<string | undefined> = computed(() => {
  let result = undefined;

  if (differenceBetweenTimestamps.value) {
    const units: HumanUnit[] = ["years", "months", "days", "hours", "minutes"];

    for (const unitsKey of units) {
      const value = examineHumanUnitBetweenTimestamps(unitsKey);
      if (value && (value >= 1 || value <= -1)) {
        result = humanizeTwoTimestamps(Math.trunc(value), unitsKey, value);
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

  <div>
    <input type="checkbox" id="compareAgainstNow" v-model="compareAgainstNow" />

    <label for="compareAgainstNow">
      Compare against now (otherwise you compare against additional timestamp)
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

  <div class="your-input" v-if="!compareAgainstNow">
    <label for="rawAdditionalInputValue">Additional timestamp to compare against</label>

    <input
      type="text"
      id="rawAdditionalInputValue"
      size="29"
      maxlength="29"
      v-model="rawAdditionalInputValue"
    />

    <span v-if="additionalInputOkay" class="indicator">✔️</span>
    <span v-else class="indicator">❗️</span>
  </div>

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

  <template v-if="!compareAgainstNow">
    <label-value-formatter
      v-if="displayInZulu"
      label="Your additional defined timestamp (Zulu)"
      :value="additionalDerivedInstant?.toString() || UNDEFINED"
      :additional-value="
        zuluZonedDateTimeAdditionalDerivedInstant?.toLocaleString(locale, optionsForLocaleString) ||
        UNDEFINED
      "
    />

    <label-value-formatter
      v-else
      :label="`Your additional defined timestamp (${browserTimeZone})`"
      :value="browserZonedDateTimeAdditionalDerivedInstant?.toString(optionsForString) || UNDEFINED"
      :additional-value="
        browserZonedDateTimeAdditionalDerivedInstant?.toLocaleString(
          locale,
          optionsForLocaleString,
        ) || UNDEFINED
      "
    />
  </template>

  <template v-if="compareAgainstNow">
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
  </template>

  <template v-if="compareAgainstNow">
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
  </template>

  <template v-if="compareAgainstNow">
    <label-value-formatter
      v-if="displayInZulu"
      label="Current week (Zulu)"
      :value="zuluZonedDateTimeNow.weekOfYear || ''"
    />

    <label-value-formatter
      v-else
      :label="`Current week (${browserTimeZone})`"
      :value="browserZonedDateTimeNow.weekOfYear || ''"
    />
  </template>

  <template v-if="compareAgainstNow">
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
  </template>

  <template v-else>
    <label-value-formatter
      label="Difference/duration between timestamps"
      :value="
        differenceBetweenTimestamps
          ?.round({ largestUnit: 'year', relativeTo: zuluZonedDateTimeNow.toString() })
          .toString() || UNDEFINED
      "
      :additional-value="differenceBetweenTimestampsHuman || UNDEFINED"
    />

    <label-value-formatter
      label="Difference in days between timestamps"
      :value="
        differenceBetweenTimestamps ? differenceBetweenTimestamps.round('day').days : UNDEFINED
      "
    />
  </template>

  <label-value-formatter
    v-if="compareAgainstNow"
    :label="`Current time (epoch in seconds)`"
    :value="instantNow.epochMilliseconds / 1000"
  />

  <label-value-formatter
    :label="`Your defined timestamp (epoch in seconds and milliseconds)`"
    :value="derivedInstant ? Math.round(derivedInstant.epochMilliseconds / 1000) : UNDEFINED"
    :additional-value="derivedInstant?.epochMilliseconds || UNDEFINED"
  />

  <label-value-formatter
    v-if="!compareAgainstNow"
    :label="`Your additional defined timestamp (epoch in seconds and milliseconds)`"
    :value="
      additionalDerivedInstant
        ? Math.round(additionalDerivedInstant.epochMilliseconds / 1000)
        : UNDEFINED
    "
    :additional-value="additionalDerivedInstant?.epochMilliseconds || UNDEFINED"
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
