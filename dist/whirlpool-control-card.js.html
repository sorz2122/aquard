// Aquard (Whirlpool/DE Fork) - based on Esirius81/aquard v0.1.0.
// Locally extended: German UI, Kalender-Vorheizen status, Heizen-auf-Zeit control.
// NOTE: this is a hand-maintained fork, not the HACS-managed original - HACS updates
// to 'aquard-card' will NOT touch this file.

const UNAVAILABLE_STATES = new Set(["unknown", "unavailable"]);
const INACTIVE_CONTROL_STATES = new Set(["off", "uit"]);
const CLIMATE_SUPPORT_TARGET_TEMPERATURE = 1;

function readEntity(hass, entityId, options = {}) {
  if (!entityId) {
    return unavailableResult("Nicht konfiguriert", "not-configured");
  }

  const stateObj = hass?.states?.[entityId];
  if (!stateObj || UNAVAILABLE_STATES.has(stateObj.state)) {
    return unavailableResult("Nicht verfuegbar", "unavailable");
  }

  if (options.numeric) {
    const number = Number(stateObj.state);
    if (!Number.isFinite(number)) {
      return unavailableResult("Nicht verfuegbar", "unavailable");
    }

    return {
      value: new Intl.NumberFormat(undefined, { maximumFractionDigits: 2 }).format(number),
      unit: stateObj.attributes?.unit_of_measurement ?? "",
      availability: "Verfuegbar",
      availabilityClass: "available",
      stateObj,
    };
  }

  return {
    value: stateObj.state,
    unit: stateObj.attributes?.unit_of_measurement ?? "",
    availability: "Verfuegbar",
    availabilityClass: "available",
    stateObj,
  };
}

function readSwitch(hass, entityId) {
  const result = readEntity(hass, entityId);
  if (result.availabilityClass !== "available") return result;

  if (result.value !== "on" && result.value !== "off") {
    return unavailableResult("Nicht verfuegbar", "unavailable");
  }

  return { ...result, value: result.value === "on" ? "An" : "Aus" };
}

function readCurrentTemperature(hass, entities = {}) {
  if (entities.water_temperature) return readEntity(hass, entities.water_temperature, { numeric: true });
  if (!entities.climate) return readEntity(hass, undefined, { numeric: true });
  const stateObj = hass?.states?.[entities.climate];
  if (!stateObj || UNAVAILABLE_STATES.has(stateObj.state)) return unavailableResult("Nicht verfuegbar", "unavailable");
  const value = Number(stateObj.attributes?.current_temperature);
  if (!Number.isFinite(value)) return unavailableResult("Nicht verfuegbar", "unavailable");
  return {
    value: new Intl.NumberFormat(undefined, { maximumFractionDigits: 2 }).format(value),
    unit: stateObj.attributes?.temperature_unit ?? "",
    availability: "Verfuegbar",
    availabilityClass: "available",
    stateObj: { ...stateObj, state: String(value) },
  };
}

function readClimate(hass, entityId) {
  const result = readEntity(hass, entityId);
  if (result.availabilityClass !== "available") return result;

  const target = Number(result.stateObj.attributes?.temperature);
  const unit = result.stateObj.attributes?.temperature_unit ?? "";
  const state = titleCase(result.value);
  const targetText = Number.isFinite(target)
    ? `${new Intl.NumberFormat(undefined, { maximumFractionDigits: 1 }).format(target)}${unit ? ` ${unit}` : ""}`
    : "Zielwert nicht verfuegbar";

  return {
    ...result,
    value: `${state} · ${targetText}`,
    targetValue: targetText,
    climateState: result.value,
    unit: "",
  };
}

function resolveTargetTemperature(hass, entityId) {
  if (!entityId || !entityId.startsWith("climate.")) return undefined;
  const stateObj = hass?.states?.[entityId];
  if (!stateObj || UNAVAILABLE_STATES.has(stateObj.state)) return undefined;
  const attributes = stateObj.attributes ?? {};
  const target = Number(attributes.temperature);
  const supportedFeatures = Number(attributes.supported_features);
  if (!Number.isFinite(target) || !Number.isFinite(supportedFeatures) || !(supportedFeatures & CLIMATE_SUPPORT_TARGET_TEMPERATURE)) return undefined;
  const configuredStep = Number(attributes.target_temp_step);
  const step = Number.isFinite(configuredStep) && configuredStep > 0 ? configuredStep : 1;
  const configuredMin = Number(attributes.min_temp);
  const configuredMax = Number(attributes.max_temp);
  return {
    entityId,
    target,
    step,
    min: Number.isFinite(configuredMin) ? configuredMin : -Infinity,
    max: Number.isFinite(configuredMax) ? configuredMax : Infinity,
    unit: attributes.temperature_unit ?? "",
    stateObj,
  };
}

function getTargetTemperatureAdjustment(control, direction) {
  if (!control || (direction !== -1 && direction !== 1)) return undefined;
  const unclamped = control.target + direction;
  const temperature = Math.min(control.max, Math.max(control.min, unclamped));
  if (Math.abs(temperature - control.target) < Number.EPSILON) return undefined;
  return { temperature, domain: "climate", service: "set_temperature", data: { entity_id: control.entityId, temperature } };
}

function formatTargetTemperature(value, unit, step = 1) {
  const decimals = Math.min(3, decimalPlaces(step));
  const formatted = new Intl.NumberFormat(undefined, { minimumFractionDigits: decimals, maximumFractionDigits: decimals }).format(value);
  return { value: formatted, unit };
}

function roundToStep(value, step) { return Number(value.toFixed(Math.min(6, decimalPlaces(step)))); }
function decimalPlaces(value) { const text = String(value); return text.includes(".") ? text.length - text.indexOf(".") - 1 : 0; }

function titleCase(value) {
  return String(value).replace(/_/g, " ").replace(/\b\w/g, (letter) => letter.toUpperCase());
}

function translateEquipmentValue(value) {
  const normalized = String(value).trim().toLowerCase();
  if (normalized === "on") return "An";
  if (normalized === "off") return "Aus";
  return titleCase(value);
}

function getControlAction(entityId, stateObj, allowSelect = false, allowClimate = false) {
  if (!entityId || !stateObj || UNAVAILABLE_STATES.has(stateObj.state)) return undefined;
  const domain = entityId.split(".", 1)[0];
  if (domain === "switch") {
    return { domain: "switch", service: "toggle", data: { entity_id: entityId } };
  }
  if (allowSelect && domain === "select") {
    return { domain: "select", service: "select_next", data: { entity_id: entityId, cycle: true } };
  }
  if (allowClimate && domain === "climate") {
    const modes = Array.isArray(stateObj.attributes?.hvac_modes) ? stateObj.attributes.hvac_modes : [];
    const currentMode = String(stateObj.state).toLowerCase();
    const hvacMode = currentMode === "off"
      ? modes.find((mode) => String(mode).toLowerCase() === "heat") ?? modes.find((mode) => String(mode).toLowerCase() !== "off")
      : modes.find((mode) => String(mode).toLowerCase() === "off");
    if (hvacMode) return { domain: "climate", service: "set_hvac_mode", data: { entity_id: entityId, hvac_mode: hvacMode } };
  }
  return undefined;
}

function isControlActive(stateObj) {
  if (!stateObj || UNAVAILABLE_STATES.has(stateObj.state)) return false;
  return !INACTIVE_CONTROL_STATES.has(String(stateObj.state).trim().toLowerCase());
}

function isHeatingActive(hass, entities = {}) {
  const heaterObj = entities.heater ? hass?.states?.[entities.heater] : undefined;
  if (heaterObj && heaterObj.state === "on") return true;
  const climateObj = entities.climate ? hass?.states?.[entities.climate] : undefined;
  if (climateObj) {
    const hvacAction = climateObj.attributes?.hvac_action;
    if (hvacAction === "heating") return true;
    if (climateObj.state === "heat") return true;
  }
  return false;
}

function unavailableResult(value, availabilityClass) {
  return {
    value,
    unit: "",
    availability: value,
    availabilityClass,
    stateObj: undefined,
  };
}

const COMPONENT_IDS = Object.freeze([
  "water_status",
  "temperature",
  "actions",
  "measurements",
  "controls",
  "details",
]);

const COMPONENT_MODES = Object.freeze(["full", "compact", "hidden"]);

const DEFAULT_COMPONENT_MODES = Object.freeze(Object.fromEntries(
  COMPONENT_IDS.map((componentId) => [componentId, "full"]),
));

function getComponentMode(config, componentId) {
  return config?.components?.[componentId] ?? DEFAULT_COMPONENT_MODES[componentId] ?? "full";
}

function isComponentVisible(config, componentId) {
  return getComponentMode(config, componentId) !== "hidden";
}

function shouldShowSensorInformation(config) {
  return config?.show_sensor_information !== false;
}

const VALID_MODES = new Set(COMPONENT_MODES);

function normalizeAquardConfig(config) {
  if (!config || typeof config !== "object" || Array.isArray(config)) {
    throw new Error("Aquard requires a configuration object");
  }

  const profile = config.profile ?? "spa";
  if (profile !== "spa") throw new Error(`Aquard does not support profile "${String(profile)}"`);

  let entities;
  if (config.entities !== undefined) {
    if (!config.entities || typeof config.entities !== "object" || Array.isArray(config.entities)) {
      throw new Error("Aquard entities must be a YAML mapping");
    }
    entities = { ...config.entities };
  } else if (typeof config.entity === "string" && config.entity.trim()) {
    entities = { water_temperature: config.entity };
  } else {
    throw new Error("Aquard requires an entities mapping");
  }

  if (!entities.auto_mode) entities.auto_mode = "input_boolean.whirlpool_auto_aktiv";
  if (!entities.manual_mode) entities.manual_mode = "input_boolean.whirlpool_manuell_heizen";
  if (!entities.time_to_ready) entities.time_to_ready = "sensor.whirlpool_zeit_bis_fertig";

  const suppliedComponents = config.components && typeof config.components === "object" && !Array.isArray(config.components)
    ? config.components
    : {};
  if (config.components !== undefined && suppliedComponents !== config.components) {
    console.warn("Aquard components must be a YAML mapping; using full component modes.");
  }

  const components = { ...suppliedComponents };
  for (const componentId of COMPONENT_IDS) {
    const suppliedMode = suppliedComponents[componentId];
    if (suppliedMode === undefined) {
      components[componentId] = DEFAULT_COMPONENT_MODES[componentId];
    } else if (VALID_MODES.has(suppliedMode)) {
      components[componentId] = suppliedMode;
    } else {
      console.warn(`Aquard component "${componentId}" has invalid mode "${String(suppliedMode)}"; using "${DEFAULT_COMPONENT_MODES[componentId]}".`);
      components[componentId] = DEFAULT_COMPONENT_MODES[componentId];
    }
  }

  return {
    ...config,
    profile,
    name: config.name || "Aquard",
    entities,
    components,
  };
}

const SPA_WATER_QUALITY_PROFILE = deepFreeze({
  essential: ["ph", "orp"],
  criticalOverrides: ["ph", "orp"],
  priority: [
    ["ph", "alert"], ["orp", "alert"], ["tds", "alert"],
    ["ph", "action_needed"], ["orp", "action_needed"], ["tds", "action_needed"],
    ["ph", "monitor"], ["orp", "monitor"], ["tds", "monitor"],
  ],
  measurements: {
    ph: {
      weight: 0.4,
      scoring: { mode: "distance", target: 7.4, curve: [[0, 100], [0.05, 99.2], [0.1, 99], [0.2, 97], [0.3, 94], [0.4, 90], [0.6, 60], [0.8, 20], [1, 0]] },
      zones: { preferred: { min: 7.35, max: 7.6 }, monitor: { min: 7.2, max: 7.8 }, action: { min: 7, max: 8 } },
      display: { min: 6.8, max: 8.2, ideal: 7.4 },
      messages: { monitorLow: "ph_slightly_below_ideal", monitorHigh: "ph_slightly_above_ideal", action_neededLow: "raise_ph", action_neededHigh: "lower_ph", alertLow: "correct_ph_before_use", alertHigh: "correct_ph_before_use" },
    },
    orp: {
      weight: 0.45,
      scoring: { mode: "shortfall", target: 730, curve: [[0, 100], [30, 98], [60, 94], [80, 90], [130, 72], [230, 40], [300, 0]] },
      zones: { preferred: { min: 650 }, monitor: { min: 600 }, action: { min: 500 } },
      display: { min: 450, max: 850, ideal: 730 },
      messages: { monitorLow: "sanitizer_performance_declining", action_neededLow: "restore_sanitizer", alertLow: "verify_sanitizer_before_use" },
    },
    ec: {
      weight: 0,
      scoring: { mode: "distance", target: 1.2, curve: [[0, 100], [0.2, 98], [0.4, 94], [0.8, 82], [1.3, 60], [1.8, 20]] },
      zones: { preferred: { min: 0.8, max: 2 }, monitor: { min: 0.6, max: 2.4 }, action: { min: 0.4, max: 3 } },
      display: { min: 0.3, max: 3.2, ideal: 1.2 }, messages: {},
    },
    tds: {
      weight: 0.15,
      scoring: { mode: "value", curve: [[0, 100], [500, 98], [1000, 94], [1500, 88], [2500, 60], [4000, 20]] },
      zones: { preferred: { max: 1500 }, monitor: { max: 2000 }, action: { max: 2500 } },
      display: { min: 0, max: 3000, ideal: 750 },
      messages: { monitorHigh: "tds_gradually_increasing", action_neededHigh: "replace_water_soon", alertHigh: "replace_water_before_use" },
    },
  },
});

const _profileMemoCache = new WeakMap();

function buildWaterQualityProfile(config) {
  const min = Number(config?.ph_preferred_min);
  const max = Number(config?.ph_preferred_max);
  if (!Number.isFinite(min) || !Number.isFinite(max) || min >= max) return SPA_WATER_QUALITY_PROFILE;

  const cached = config && _profileMemoCache.get(config);
  if (cached && cached.min === min && cached.max === max) return cached.profile;

  const center = (min + max) / 2;
  const monitorPad = 0.2;
  const actionPad = 0.6;
  const displayPad = 1.4;
  const profile = {
    ...SPA_WATER_QUALITY_PROFILE,
    measurements: {
      ...SPA_WATER_QUALITY_PROFILE.measurements,
      ph: {
        ...SPA_WATER_QUALITY_PROFILE.measurements.ph,
        scoring: { ...SPA_WATER_QUALITY_PROFILE.measurements.ph.scoring, target: center },
        zones: {
          preferred: { min, max },
          monitor: { min: min - monitorPad, max: max + monitorPad },
          action: { min: min - actionPad, max: max + actionPad },
        },
        display: { min: min - displayPad, max: max + displayPad, ideal: center },
      },
    },
  };
  if (config) _profileMemoCache.set(config, { min, max, profile });
  return profile;
}

function evaluateSpaWaterQuality(values, profile = SPA_WATER_QUALITY_PROFILE) { return evaluateWaterQuality(values, profile); }

function evaluateWaterQuality(values, profile) {
  const measurements = Object.fromEntries(Object.entries(profile.measurements).map(([key, definition]) => [key, evaluateMeasurement(values[key], definition)]));
  const configuredKeys = Object.keys(profile.measurements).filter((key) => Object.hasOwn(values, key));
  if (configuredKeys.some((key) => !measurements[key]?.available)) return result("unknown", null, null, "sensor_data_unavailable", measurements);

  const weightedKeys = configuredKeys.filter((key) => measurements[key].available && profile.measurements[key].weight > 0);
  const totalWeight = weightedKeys.reduce((sum, key) => sum + profile.measurements[key].weight, 0);
  if (!totalWeight) return result("unknown", null, null, "sensor_data_unavailable", measurements);
  const score = Math.round(weightedKeys.reduce((sum, key) => sum + measurements[key].score * profile.measurements[key].weight, 0) / totalWeight);
  const primaryIssue = findPrimaryIssue(measurements, profile.priority);
  const severities = weightedKeys.map((key) => measurements[key].severity);
  const status = ["alert", "action_needed", "monitor"].find((severity) => severities.includes(severity)) ?? "excellent";
  const defaultMessages = { excellent: "enjoy_your_spa", monitor: "keep_monitoring", action_needed: "maintenance_due", alert: "check_water_before_use" };
  return result(status, score, primaryIssue?.measurement ?? null, primaryIssue?.messageKey ?? defaultMessages[status], measurements);
}

function computeRangeColor(direction, severity, intensity) {
  if (direction !== "low" && direction !== "high") return null;
  const severityBase = { monitor: 0, action_needed: 0.34, alert: 0.67 }[severity] ?? 0.34;
  const badness = Math.min(1, severityBase + (Math.max(0, Math.min(1, intensity)) * 0.34));
  const hue = 34 - (badness * 34); // 34 = orange, 0 = red
  return `hsl(${hue.toFixed(0)} 86% 54%)`;
}

function evaluateMeasurement(input, definition) {
  const value = finiteNumber(input);
  if (value === undefined) return unavailableMeasurement();
  const classification = classify(value, definition.zones);
  const score = interpolateCurve(scoringInput(value, definition.scoring), definition.scoring.curve);
  return { available: true, value, score: Math.round(score), severity: classification.severity, messageKey: classification.severity === "excellent" ? null : definition.messages[`${classification.severity}${classification.side}`] ?? null, range: rangePresentation(value, classification, definition) };
}

function classify(value, zones) {
  if (outside(value, zones.action)) return { severity: "alert", side: side(value, zones.action) };
  if (outside(value, zones.monitor)) return { severity: "action_needed", side: side(value, zones.monitor) };
  if (outside(value, zones.preferred)) return { severity: "monitor", side: side(value, zones.preferred) };
  return { severity: "excellent", side: "" };
}
function outside(value, zone) { return (zone.min !== undefined && value < zone.min) || (zone.max !== undefined && value > zone.max); }
function side(value, zone) { return zone.min !== undefined && value < zone.min ? "Low" : "High"; }

function rangePresentation(value, classification, definition) {
  const { display, zones } = definition;
  const currentPosition = percentage(value, display.min, display.max);
  const idealPosition = percentage(display.ideal, display.min, display.max);
  if (classification.severity === "excellent") return { direction: "ideal", intensity: 0, currentPosition, idealPosition };
  const isLow = classification.side === "Low";
  const boundary = isLow ? zones.preferred.min : zones.preferred.max;
  const extreme = isLow ? zones.action.min ?? display.min : zones.action.max ?? display.max;
  const intensity = Math.min(1, Math.abs(value - boundary) / Math.max(Math.abs(extreme - boundary), Number.EPSILON));
  return { direction: isLow ? "low" : "high", intensity, currentPosition, idealPosition };
}
function percentage(value, min, max) { return Math.max(0, Math.min(100, ((value - min) / (max - min)) * 100)); }
function scoringInput(value, scoring) { if (scoring.mode === "distance") return Math.abs(value - scoring.target); if (scoring.mode === "shortfall") return Math.max(0, scoring.target - value); return value; }
function interpolateCurve(value, curve) { if (value <= curve[0][0]) return curve[0][1]; for (let index = 1; index < curve.length; index += 1) { const [endValue, endScore] = curve[index]; if (value <= endValue) { const [startValue, startScore] = curve[index - 1]; return startScore + ((value - startValue) / (endValue - startValue)) * (endScore - startScore); } } return curve[curve.length - 1][1]; }
function findPrimaryIssue(measurements, priority) { for (const [key, severity] of priority) if (measurements[key]?.severity === severity) return { measurement: key, messageKey: measurements[key].messageKey }; return null; }
function result(status, score, primaryIssue, messageKey, measurements) { return { status, canUse: status === "excellent" || status === "monitor" ? true : status === "unknown" ? null : false, score, primaryIssue, messageKey, measurements }; }
function unavailableMeasurement() { return { available: false, value: null, score: null, severity: "unknown", messageKey: "sensor_data_unavailable" }; }
function finiteNumber(value) { if (value === null || value === undefined || value === "" || value === "unknown" || value === "unavailable") return undefined; const number = Number(value); return Number.isFinite(number) ? number : undefined; }
function deepFreeze(value) { Object.freeze(value); for (const child of Object.values(value)) if (child && typeof child === "object" && !Object.isFrozen(child)) deepFreeze(child); return value; }

const DEFAULT_TEMPERATURE_GAUGE_PROFILE = Object.freeze({
  min: 0,
  max: 40,
  arcFraction: 0.75,
});

const TEMPERATURE_GAUGE_GEOMETRY = Object.freeze({
  center: 80,
  radius: 72,
  rotation: 135,
});

function temperatureToArc(value, profile = DEFAULT_TEMPERATURE_GAUGE_PROFILE) {
  const numericValue = Number(value);
  const clampedValue = Number.isFinite(numericValue)
    ? Math.min(profile.max, Math.max(profile.min, numericValue))
    : profile.min;
  const progress = (clampedValue - profile.min) / (profile.max - profile.min);
  const circumference = 2 * Math.PI * TEMPERATURE_GAUGE_GEOMETRY.radius;
  const arcLength = circumference * profile.arcFraction;
  const markerAngle = TEMPERATURE_GAUGE_GEOMETRY.rotation + (360 * profile.arcFraction * progress);
  const markerRadians = markerAngle * (Math.PI / 180);
  return {
    clampedValue,
    progress,
    percentage: progress * 100,
    circumference,
    arcLength,
    progressLength: arcLength * progress,
    marker: {
      x: TEMPERATURE_GAUGE_GEOMETRY.center + (TEMPERATURE_GAUGE_GEOMETRY.radius * Math.cos(markerRadians)),
      y: TEMPERATURE_GAUGE_GEOMETRY.center + (TEMPERATURE_GAUGE_GEOMETRY.radius * Math.sin(markerRadians)),
    },
  };
}

function renderTemperatureGauge(value, profile = DEFAULT_TEMPERATURE_GAUGE_PROFILE, targetValue = null, isHeating = false) {
  const arc = temperatureToArc(value, profile);
  const remainingProgress = arc.circumference - arc.progressLength;
  const remainingTrack = arc.circumference - arc.arcLength;
  const { center, radius, rotation } = TEMPERATURE_GAUGE_GEOMETRY;

  const hasTarget = Number.isFinite(Number(targetValue));
  let targetMarkup = "";
  if (hasTarget) {
    const targetArc = temperatureToArc(targetValue, profile);
    const labelRadius = radius + 17;
    const labelAngleDeg = rotation + (360 * profile.arcFraction * targetArc.progress);
    const labelAngleRad = labelAngleDeg * (Math.PI / 180);
    const labelX = center + (labelRadius * Math.cos(labelAngleRad));
    const labelY = center + (labelRadius * Math.sin(labelAngleRad));
    targetMarkup = `
      <circle class="temperature-gauge-target-marker" cx="${targetArc.marker.x}" cy="${targetArc.marker.y}" r="5"/>
      <text class="temperature-gauge-target-label" x="${labelX}" y="${labelY}" text-anchor="middle" dominant-baseline="middle">${Math.round(Number(targetValue))}&deg;</text>`;
  }

  const heatingBubbles = `
    <g class="heating-bubbles">
      <circle class="heat-bubble hb1" cx="70" cy="112" r="4"/>
      <circle class="heat-bubble hb2" cx="86" cy="118" r="3"/>
      <circle class="heat-bubble hb3" cx="78" cy="104" r="2.4"/>
      <circle class="heat-bubble hb4" cx="92" cy="108" r="3.4"/>
      <circle class="heat-bubble hb5" cx="66" cy="98" r="2"/>
    </g>`;

  return `
    <svg class="temperature-gauge-svg${isHeating ? " is-heating" : ""}" viewBox="0 0 160 160" role="img" aria-label="Temperaturskala ${Math.round(arc.percentage)} Prozent">
      <defs>
        <linearGradient id="aquard-droplet" x1="28%" y1="18%" x2="72%" y2="88%">
          <stop offset="0" stop-color="var(--aq-gauge-drop-light)"/>
          <stop offset=".48" stop-color="var(--aq-gauge-drop-mid)"/>
          <stop offset="1" stop-color="var(--aq-gauge-drop-deep)"/>
        </linearGradient>
        <linearGradient id="aquard-droplet-highlight" x1="35%" y1="20%" x2="62%" y2="75%">
          <stop offset="0" stop-color="var(--aq-gauge-highlight)" stop-opacity=".9"/>
          <stop offset="1" stop-color="var(--aq-gauge-highlight)" stop-opacity="0"/>
        </linearGradient>
        <filter id="aquard-droplet-glow" x="-60%" y="-60%" width="220%" height="220%">
          <feGaussianBlur stdDeviation="5" result="blur"/>
          <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
        </filter>
        <linearGradient id="aquard-heat-progress" x1="0%" y1="100%" x2="100%" y2="0%">
          <stop offset="0%" stop-color="#ff3b3b"/>
          <stop offset="50%" stop-color="#ff7b00"/>
          <stop offset="100%" stop-color="#ffb74d"/>
        </linearGradient>
      </defs>
      <circle class="temperature-gauge-track" cx="${center}" cy="${center}" r="${radius}"
        transform="rotate(${rotation} ${center} ${center})"
        stroke-dasharray="${arc.arcLength} ${remainingTrack}"/>
      <circle class="temperature-gauge-progress" cx="${center}" cy="${center}" r="${radius}"
        transform="rotate(${rotation} ${center} ${center})"
        stroke-dasharray="${arc.progressLength} ${remainingProgress}"/>
      <circle class="temperature-gauge-marker" cx="${arc.marker.x}" cy="${arc.marker.y}" r="3.5"/>
      ${targetMarkup}
      <g class="temperature-gauge-droplet" filter="url(#aquard-droplet-glow)">
        <path d="M80 29 C77 39 51 72 51 98 C51 116 64 130 80 130 C96 130 109 116 109 98 C109 72 83 39 80 29Z" fill="url(#aquard-droplet)" fill-opacity=".88"/>
        <path d="M72 54 C63 70 58 84 59 98 C60 108 65 116 72 121 C67 107 67 86 72 54Z" fill="url(#aquard-droplet-highlight)" opacity=".76"/>
        <ellipse cx="70" cy="72" rx="5" ry="9" fill="var(--aq-gauge-highlight)" opacity=".68" transform="rotate(24 70 72)"/>
        <path class="temperature-gauge-drop-edge" d="M80 29 C77 39 51 72 51 98 C51 116 64 130 80 130 C96 130 109 116 109 98 C109 72 83 39 80 29Z"/>
        ${isHeating ? heatingBubbles : ""}
      </g>
    </svg>`;
}

const STATUS_INDICATOR_GEOMETRY = Object.freeze({ viewBox: "0 0 160 160", center: 80, radius: 58, strokeWidth: 8 });

const STATUS_SYMBOLS = Object.freeze({
  excellent: '<path class="status-symbol" d="M53 81 L72 100 L109 61"/>',
  monitor: '<path class="status-symbol" d="M43 80 C55 61 69 53 80 53 C91 53 105 61 117 80 C105 99 91 107 80 107 C69 107 55 99 43 80Z"/><circle class="status-symbol" cx="80" cy="80" r="12"/>',
  action_needed: '<path class="status-symbol" d="M80 48 L80 88"/><circle class="status-symbol-fill" cx="80" cy="108" r="5"/>',
  alert: '<path class="status-symbol" d="M80 48 L80 88"/><circle class="status-symbol-fill" cx="80" cy="108" r="5"/>',
  unknown: '<path class="status-symbol" d="M62 64 C65 50 76 44 87 46 C101 48 107 60 102 72 C98 82 84 84 81 94 L81 98"/><circle class="status-symbol-fill" cx="81" cy="112" r="4"/>',
});

function renderStatusIndicator(status) {
  const geometry = STATUS_INDICATOR_GEOMETRY;
  return `<svg class="status-indicator-svg" viewBox="${geometry.viewBox}" aria-hidden="true">
    <defs><filter id="aquard-status-glow" x="-60%" y="-60%" width="220%" height="220%"><feGaussianBlur stdDeviation="4" result="glow"/><feMerge><feMergeNode in="glow"/><feMergeNode in="SourceGraphic"/></feMerge></filter></defs>
    <circle class="status-ring" cx="${geometry.center}" cy="${geometry.center}" r="${geometry.radius}" stroke-width="${geometry.strokeWidth}"/>
    <path class="status-ring-highlight" d="M39 60 A58 58 0 0 1 121 60"/>
    <path class="status-ring-reflection" d="M45 111 A58 58 0 0 0 63 132"/>
    <g class="status-symbol-group">${STATUS_SYMBOLS[status] ?? STATUS_SYMBOLS.unknown}</g>
    <g class="status-sparkle"><path d="M42 35 L42 46 M36.5 40.5 L47.5 40.5"/><circle cx="53" cy="30" r="2"/></g>
  </svg>`;
}

function renderTargetArrow(direction) {
  const isLeft = direction === "decrease";
  const points = isLeft ? "31 19 20 28 31 37" : "25 19 36 28 25 37";
  return `<svg class="target-arrow-svg" viewBox="0 0 56 56" aria-hidden="true">
    <path class="target-button-glass" d="M13 5 H43 Q51 5 51 13 V43 Q51 51 43 51 H13 Q5 51 5 43 V13 Q5 5 13 5Z"/>
    <path class="target-button-highlight" d="M13 8 H43 Q47 8 48 12"/>
    <polyline class="target-arrow-chevron" points="${points}"/>
  </svg>`;
}


function renderWaterStatus({ status, mode, actions = "" }) {
  if (mode === "hidden") return "";
  if (mode === "compact") {
    return `<section class="aquard-component aquard-component--compact status-panel status-panel--compact status-${status}" data-component="water_status">
      <div class="status-orb">${renderStatusIndicator(status)}</div>
      <div class="hero-copy"><div class="status-headline"></div><div class="status-score"></div></div>
      ${actions}
    </section>`;
  }
  return `<section class="aquard-component aquard-component--full hero-panel status-panel status-${status}" data-component="water_status">
    <div class="status-orb">${renderStatusIndicator(status)}</div>
    <div class="hero-copy"><div class="status-headline"></div></div>
    ${actions}
  </section>`;
}


function renderTimeToReadyText(value) {
  if (!value || UNAVAILABLE_STATES.has(String(value))) return null;
  const str = String(value).trim();
  const num = parseFloat(str);
  if (Number.isFinite(num)) {
    const h = Math.floor(num);
    const m = Math.round((num - h) * 60);
    if (h > 0) return `Zieltemperatur erreicht in ${h} h ${String(m).padStart(2, "0")} m`;
    return `Zieltemperatur erreicht in ${m} Min`;
  }
  return str;
}

function renderTemperature({ mode, reading, targetControl = "", configured = true, targetValue = null, isHeating = false, timeToReady = null }) {
  if (mode === "hidden" || !configured) return "";
  const timeToReadyMarkup = isHeating && timeToReady
    ? `<div class="time-to-ready-badge" title="Dauer bis zur Zieltemperatur"><ha-icon icon="mdi:thermometer-chevron-up"></ha-icon><span>${timeToReady}</span></div>`
    : "";
  if (mode === "compact") {
    return `<section class="aquard-component aquard-component--compact temperature-panel temperature-panel--compact ${reading.availabilityClass}" data-component="temperature">
      <div class="temperature-copy"><div class="section-label temperature-label"></div><div class="temperature-reading"><span class="temperature-value"><span class="temperature-whole"></span><span class="temperature-decimal"></span></span><span class="temperature-unit"></span></div>${timeToReadyMarkup}${targetControl}</div>
    </section>`;
  }
  return `<section class="aquard-component aquard-component--full hero-panel temperature-panel ${reading.availabilityClass}" data-component="temperature">
    <div class="temperature-copy"><div class="section-label temperature-label"></div><div class="temperature-reading"><span class="temperature-value"><span class="temperature-whole"></span><span class="temperature-decimal"></span></span><span class="temperature-unit"></span></div>${timeToReadyMarkup}${targetControl}</div>
    <div class="temperature-gauge">${renderTemperatureGauge(reading.stateObj?.state, DEFAULT_TEMPERATURE_GAUGE_PROFILE, targetValue, isHeating)}</div>
  </section>`;
}

function renderActions({ mode, hasStatus, standalone = false }) {
  if (mode === "hidden" || !hasStatus) return "";
  if (mode === "compact") {
    return `<aside class="aquard-component aquard-component--compact status-summary status-summary--compact${standalone ? " actions-standalone" : ""}" data-component="actions"><div class="status-action"></div></aside>`;
  }
  return `<aside class="aquard-component aquard-component--full status-summary${standalone ? " actions-standalone" : ""}" data-component="actions"><div class="status-action"></div><div class="status-support"><span class="status-dot"></span><span class="status-support-text"></span></div></aside>`;
}

function renderMeasurements({ mode, hasMeasurements }) {
  if (mode === "hidden" || !hasMeasurements) return "";
  return `<section class="aquard-component aquard-component--${mode} measurement-section${mode === "compact" ? " measurement-section--compact" : ""}" data-component="measurements" aria-label="Wasserqualitaetsmesswerte"><div class="metric-grid"></div></section>`;
}

function renderControls({ mode, hasControls }) {
  if (mode === "hidden" || !hasControls) return "";
  return `<section class="aquard-component aquard-component--${mode} equipment-section${mode === "compact" ? " equipment-section--compact" : ""}" data-component="controls" aria-label="Geraetestatus"><div class="equipment-grid"></div></section>`;
}

function renderDetails({ mode, name, availabilityClass, showAvailability = true, autoActive = false, manualActive = false, showModeSwitch = false }) {
  if (mode === "hidden") return "";
  return `<header class="aquard-component aquard-component--${mode} aquard-header${mode === "compact" ? " aquard-header--compact" : ""}" data-component="details">
    <div class="brand-lockup"><div><div class="brand-name"></div><div class="brand-context"></div></div><span class="brand-mark" aria-hidden="true"><ha-icon icon="mdi:waves"></ha-icon></span></div>
    <div class="header-right-controls">
      ${showModeSwitch ? `<div class="mode-switch-pill" role="group" aria-label="Betriebsmodus">
        <button type="button" class="mode-pill-btn ${autoActive ? "active" : ""}" data-mode-action="auto">Auto</button>
        <button type="button" class="mode-pill-btn ${manualActive ? "active" : ""}" data-mode-action="manual">Manuell</button>
      </div>` : ""}
      ${mode === "full" && showAvailability ? `<div class="header-availability ${availabilityClass}"><span class="status-dot"></span><span class="header-availability-text"></span></div>` : ""}
    </div>
  </header>`;
}


// ---------------------------------------------------------------------------
// Whirlpool extensions: calendar preheat status + timed heating control
// ---------------------------------------------------------------------------

function formatDateTimeDE(isoOrDate) {
  if (!isoOrDate) return null;
  const date = isoOrDate instanceof Date ? isoOrDate : new Date(isoOrDate);
  if (Number.isNaN(date.getTime())) return null;
  return date.toLocaleString("de-DE", { weekday: "short", day: "2-digit", month: "2-digit", hour: "2-digit", minute: "2-digit" });
}

function readPreheat(hass, entities = {}) {
  const neededObj = entities.preheat_needed ? hass?.states?.[entities.preheat_needed] : undefined;
  const startTimeObj = entities.preheat_start_time ? hass?.states?.[entities.preheat_start_time] : undefined;
  const calendarObj = entities.calendar ? hass?.states?.[entities.calendar] : undefined;

  const neededState = neededObj && !UNAVAILABLE_STATES.has(neededObj.state) ? neededObj.state : undefined;
  const needed = neededState === "on" ? true : neededState === "off" ? false : undefined;

  const startTimeRaw = startTimeObj && !UNAVAILABLE_STATES.has(startTimeObj.state) ? startTimeObj.state : undefined;
  const startTimeFormatted = startTimeRaw ? formatDateTimeDE(startTimeRaw) : null;

  const appointmentStart = calendarObj?.attributes?.start_time && !UNAVAILABLE_STATES.has(calendarObj.state)
    ? calendarObj.attributes.start_time
    : undefined;
  const appointmentTitle = calendarObj?.attributes?.message ?? null;
  const appointmentFormatted = appointmentStart ? formatDateTimeDE(appointmentStart) : null;

  return { needed, startTimeFormatted, appointmentFormatted, appointmentTitle, hasAnyEntity: Boolean(entities.preheat_needed || entities.preheat_start_time || entities.calendar) };
}

function readHeatTimer(hass, entityId) {
  if (!entityId) return { configured: false, active: false, paused: false, remainingSeconds: 0, durationSeconds: 0 };
  const stateObj = hass?.states?.[entityId];
  if (!stateObj || UNAVAILABLE_STATES.has(stateObj.state)) {
    return { configured: true, active: false, paused: false, remainingSeconds: 0, durationSeconds: 0 };
  }
  const parseHMS = (text) => {
    if (!text) return 0;
    const parts = String(text).split(":").map(Number);
    if (parts.length !== 3 || parts.some((n) => !Number.isFinite(n))) return 0;
    return (parts[0] * 3600) + (parts[1] * 60) + parts[2];
  };
  const durationSeconds = parseHMS(stateObj.attributes?.duration);
  if (stateObj.state === "active") {
    const finishesAt = stateObj.attributes?.finishes_at ? new Date(stateObj.attributes.finishes_at).getTime() : null;
    const remainingSeconds = finishesAt ? Math.max(0, Math.round((finishesAt - Date.now()) / 1000)) : 0;
    return { configured: true, active: true, paused: false, remainingSeconds, durationSeconds };
  }
  if (stateObj.state === "paused") {
    return { configured: true, active: false, paused: true, remainingSeconds: parseHMS(stateObj.attributes?.remaining), durationSeconds };
  }
  return { configured: true, active: false, paused: false, remainingSeconds: 0, durationSeconds };
}

function formatRemaining(totalSeconds) {
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;
  return `${String(minutes).padStart(2, "0")}:${String(seconds).padStart(2, "0")}`;
}

function renderPreheat({ entities, preheat, title = UI_TEXT.preheatTitle }) {
  if (!preheat.hasAnyEntity) return "";

  const noAppointment = Boolean(entities.calendar) && !preheat.appointmentFormatted;
  if (noAppointment) {
    return `<section class="aquard-component aquard-component--full wp-panel wp-preheat-panel" data-component="preheat">
      <div class="wp-panel-header"><ha-icon icon="mdi:calendar-clock" class="wp-panel-icon"></ha-icon><span class="wp-panel-title">${title}</span><span class="wp-status-pill">${UI_TEXT.preheatNoAppointment}</span></div>
    </section>`;
  }

  const neededLabel = preheat.needed === true ? UI_TEXT.preheatNeededYes : preheat.needed === false ? UI_TEXT.preheatNeededNo : UI_TEXT.preheatNeededUnknown;
  const neededClass = preheat.needed === true ? "available" : preheat.needed === false ? "" : "unavailable";
  const appointmentLine = entities.calendar
    ? `<div class="wp-row"><span class="wp-row-label">${UI_TEXT.preheatNextAppointment}</span><span class="wp-row-value">${preheat.appointmentTitle ? `${preheat.appointmentFormatted} &middot; ${preheat.appointmentTitle}` : preheat.appointmentFormatted}</span></div>`
    : "";
  const startTimeLine = entities.preheat_start_time
    ? `<div class="wp-row"><span class="wp-row-label">${UI_TEXT.preheatStartTime}</span><span class="wp-row-value">${preheat.startTimeFormatted ?? UI_TEXT.preheatStartTimeUnknown}</span></div>`
    : "";
  return `<section class="aquard-component aquard-component--full wp-panel wp-preheat-panel" data-component="preheat">
    <div class="wp-panel-header"><ha-icon icon="mdi:calendar-clock" class="wp-panel-icon"></ha-icon><span class="wp-panel-title">${title}</span><span class="wp-status-pill ${neededClass}">${neededLabel}</span></div>
    <div class="wp-panel-body">${appointmentLine}${startTimeLine}</div>
  </section>`;
}

function computeDosing(hass, entities, config) {
  if (!entities.ph || !entities.orp) return { hasEntities: false };
  const phState = hass?.states?.[entities.ph];
  const orpState = hass?.states?.[entities.orp];
  const ph = phState ? Number(phState.state) : NaN;
  const orp = orpState ? Number(orpState.state) : NaN;
  if (!Number.isFinite(ph) || !Number.isFinite(orp)) return { hasEntities: true, available: false };

  // Formula ported from whirlpool-water-card (Esirius81 / user's existing dosing card).
  const volumeL = Number(config?.volume_l) > 0 ? Number(config.volume_l) : 650;
  const volumeM3 = volumeL / 1000;
  const targetPh = Number.isFinite(Number(config?.ph_preferred_min)) && Number.isFinite(Number(config?.ph_preferred_max))
    ? (Number(config.ph_preferred_min) + Number(config.ph_preferred_max)) / 2
    : (Number(config?.dosing_target_ph) || 7.3);
  const targetFc = Number(config?.dosing_target_fc) || 0.6;
  const orpTarget = Number(config?.dosing_orp_target) || 675;
  const orpWarn = Number(config?.dosing_orp_warn) || 650;
  const minGram = Number(config?.dosing_min_gram) || 5;
  const minGramChlor = Number(config?.dosing_min_gram_chlor) || 0.5;

  const dphMinus = ph - targetPh;
  const dphPlus = targetPh - ph;
  const phMinusG = dphMinus > 0 ? (dphMinus / 0.1) * 100 * (volumeM3 / 10) : 0;
  const phPlusG = dphPlus > 0 ? (dphPlus / 0.1) * 100 * (volumeM3 / 10) : 0;

  let fcEst;
  if (ph <= 7.1) fcEst = 0.5 + (orp - 715) / 18;
  else if (ph <= 7.3) fcEst = 0.5 + (orp - 703) / 18;
  else if (ph <= 7.5) fcEst = 0.5 + (orp - 691) / 18;
  else if (ph <= 7.7) fcEst = 0.5 + (orp - 679) / 18;
  else fcEst = 0.5 + (orp - 667) / 18;
  fcEst = Math.max(fcEst, 0);
  const deltaFc = Math.max(targetFc - fcEst, 0);
  const chlorG = orp < orpWarn ? (deltaFc * volumeL) / 560 : 0;

  const showPhPlus = phPlusG >= minGram;
  const showPhMinus = phMinusG >= minGram;
  const showChlor = chlorG >= minGramChlor;

  return {
    hasEntities: true,
    available: true,
    needed: showPhPlus || showPhMinus || showChlor,
    showPhPlus, showPhMinus, showChlor,
    phPlusG, phMinusG, chlorG,
  };
}

function buildDosingCornerBadge(key, dosing) {
  if (!dosing?.available) return "";
  if (key === "ph") {
    if (dosing.showPhPlus) return `<span class="metric-corner-badge" title="${UI_TEXT.dosingPhPlus}"><ha-icon icon="mdi:cup-outline"></ha-icon>+${dosing.phPlusG.toFixed(1)}g</span>`;
    if (dosing.showPhMinus) return `<span class="metric-corner-badge" title="${UI_TEXT.dosingPhMinus}"><ha-icon icon="mdi:cup-outline"></ha-icon>-${dosing.phMinusG.toFixed(1)}g</span>`;
    return "";
  }
  if (key === "orp") {
    if (dosing.showChlor) return `<span class="metric-corner-badge" title="${UI_TEXT.dosingChlor}"><ha-icon icon="mdi:bottle-tonic-outline"></ha-icon>+${dosing.chlorG.toFixed(1)}g</span>`;
    return "";
  }
  return "";
}

function renderTimedHeating({ entities, timer, durationValue, title = UI_TEXT.timedHeatingTitle }) {
  if (!entities.heat_timer && !entities.heat_start) return "";
  const running = timer.active || timer.paused;
  const statusLabel = running ? UI_TEXT.timedHeatingActive : UI_TEXT.timedHeatingIdle;
  const statusClass = running ? "available" : "";
  const body = running
    ? `<div class="wp-row"><span class="wp-row-label">${UI_TEXT.timedHeatingRemaining}</span><span class="wp-row-value wp-countdown">${formatRemaining(timer.remainingSeconds)}</span></div>
       <button type="button" class="wp-button wp-button-stop" data-wp-action="stop">${UI_TEXT.timedHeatingStop}</button>`
    : `<div class="wp-row wp-duration-row"><span class="wp-row-label">${UI_TEXT.timedHeatingDuration}</span>
         <div class="wp-duration-control">
           <button type="button" class="wp-stepper" data-wp-action="dec" aria-label="-10 ${UI_TEXT.timedHeatingMinutes}">-</button>
           <span class="wp-row-value wp-duration-value">${durationValue !== null ? `${durationValue} ${UI_TEXT.timedHeatingMinutes}` : "--"}</span>
           <button type="button" class="wp-stepper" data-wp-action="inc" aria-label="+10 ${UI_TEXT.timedHeatingMinutes}">+</button>
         </div>
       </div>
       <button type="button" class="wp-button wp-button-start" data-wp-action="start" ${entities.heat_start ? "" : "disabled"}>${UI_TEXT.timedHeatingStart}</button>`;
  return `<section class="aquard-component aquard-component--full wp-panel wp-heating-panel" data-component="timed_heating">
    <div class="wp-panel-header"><ha-icon icon="mdi:timer-outline" class="wp-panel-icon"></ha-icon><span class="wp-panel-title">${title}</span><span class="wp-status-pill ${statusClass}">${statusLabel}</span></div>
    <div class="wp-panel-body">${body}</div>
  </section>`;
}

const styles = `
  :host {
    display:block; width:100%; min-width:0; max-width:none; box-sizing:border-box; container-type:inline-size;
    --aq-bg:#04131e; --aq-surface:rgba(7,31,45,.9); --aq-border:rgba(55,190,222,.22);
    --aq-text:#f1f8fb; --aq-muted:#83a7b9; --aq-blue:#18c8f3; --aq-green:#43e66c; --aq-yellow:#f1d45b; --aq-orange:#f0a04b; --aq-amber:#f0b56b; --aq-red:#ff6474; --aq-water-line:var(--aq-blue);
    --aq-gauge-track:rgba(24,200,243,.18); --aq-gauge-progress:var(--aq-blue); --aq-gauge-marker:#fff; --aq-gauge-drop-light:#8cecff; --aq-gauge-drop-mid:#18c8f3; --aq-gauge-drop-deep:#087cab; --aq-gauge-highlight:#f1fdff;
    --aq-xl:26px; --aq-lg:18px; --aq-md:16px; --aq-gap:clamp(12px,1.5cqw,16px);
    --aq-pad:clamp(16px,2.2cqw,24px); --aq-motion:320ms cubic-bezier(.22,1,.36,1);
  }
  *,*::before,*::after{box-sizing:border-box}
  ha-card{display:block;width:100%;min-width:0;max-width:none;overflow:hidden;padding:var(--aq-pad);border:1px solid var(--aq-border);border-radius:var(--aq-xl);color:var(--aq-text);background:radial-gradient(circle at 12% 8%,rgba(13,166,199,.12),transparent 32%),radial-gradient(circle at 86% 22%,rgba(20,116,173,.14),transparent 34%),linear-gradient(145deg,#071b28,var(--aq-bg) 58%,#030e17);box-shadow:0 24px 64px rgba(0,0,0,.34),inset 0 1px rgba(255,255,255,.03)}
  .setup-card{min-height:140px}.setup-state{display:flex;min-height:108px;align-items:center;justify-content:center;gap:16px;padding:18px;text-align:left}.setup-state ha-icon{width:38px;height:38px;flex:0 0 auto;color:var(--aq-blue);--mdc-icon-size:38px}.setup-state h2{margin:0 0 6px;font-size:1.15rem}.setup-state p{margin:0;color:var(--aq-muted);line-height:1.45}
  main,.aquard-header,.hero-grid,.hero-panel,.measurement-section,.metric-grid,.equipment-section,.equipment-grid{width:100%;min-width:0}
  .aquard-header{display:flex;align-items:center;justify-content:space-between;gap:18px;margin-bottom:clamp(8px,1cqw,12px);padding:0 4px}
  .brand-lockup,.header-availability,.status-support,.climate-line,.metric-state{display:flex;align-items:center}
  .brand-lockup{min-width:0;gap:8px}.brand-mark{display:grid;width:30px;height:30px;flex:0 0 auto;place-items:center;color:var(--aq-blue);filter:drop-shadow(0 0 10px rgba(25,199,242,.32))}.brand-mark ha-icon{width:29px;height:29px;--mdc-icon-size:29px}
  .brand-name{overflow-wrap:anywhere;font-size:clamp(1.45rem,3.2cqw,1.9rem);font-weight:650;letter-spacing:-.025em;line-height:1}.brand-context{display:none}.header-availability,.metric-state{color:var(--aq-muted);font-size:.72rem}.header-availability{flex:0 0 auto;gap:8px}
  .header-right-controls{display:flex;align-items:center;gap:12px;flex:0 0 auto}
  .mode-switch-pill{display:inline-flex;align-items:center;padding:3px;border-radius:999px;border:1px solid var(--aq-border);background:rgba(5,24,37,.8);box-shadow:inset 0 1px 3px rgba(0,0,0,.4)}
  .mode-pill-btn{border:none;background:transparent;color:var(--aq-muted);padding:4px 12px;border-radius:999px;font-size:.74rem;font-weight:650;letter-spacing:.02em;cursor:pointer;transition:all 180ms ease}
  .mode-pill-btn:hover{color:var(--aq-text)}
  .mode-pill-btn.active{background:linear-gradient(145deg,var(--aq-blue),#087cab);color:#fff;box-shadow:0 0 10px rgba(24,200,243,.4)}
  .time-to-ready-badge{display:inline-flex;align-items:center;gap:6px;padding:5px 12px;margin-bottom:6px;border-radius:999px;border:1px solid rgba(255,183,77,.5);background:rgba(255,123,0,.15);color:#ffb74d;font-size:.78rem;font-weight:600;white-space:normal;line-height:1.2}
  .time-to-ready-badge ha-icon{--mdc-icon-size:16px;width:16px;height:16px;flex:0 0 auto}
  .status-dot{width:8px;height:8px;flex:0 0 auto;border-radius:50%;background:#617987}.available .status-dot{background:var(--aq-green);box-shadow:0 0 12px rgba(67,230,108,.7)}.unavailable .status-dot{background:var(--aq-amber);box-shadow:0 0 10px rgba(240,181,107,.35)}
  .hero-grid{position:relative;display:grid;grid-template-columns:minmax(0,1.08fr) minmax(0,.92fr);gap:var(--aq-gap);isolation:isolate}.hero-water-line{position:absolute;inset:0;z-index:0;width:100%;height:100%;overflow:visible;color:var(--aq-water-line);opacity:.22;filter:drop-shadow(0 0 7px color-mix(in srgb,currentColor 48%,transparent));pointer-events:none}.hero-water-line path{fill:none;stroke:currentColor}.hero-water-line .water-ribbon{fill:currentColor;stroke:none;opacity:.16}.hero-water-line .water-ribbon-secondary{opacity:.09}.hero-water-line .water-wave{stroke-width:1.5}.hero-water-line .water-wave-primary{stroke-width:2.5;filter:drop-shadow(0 0 7px currentColor)}.hero-water-line .water-line-secondary{stroke-width:1;opacity:.6}.hero-water-line .water-filament{stroke-width:.65;opacity:.46}.hero-water-line .water-bubbles{fill:none;stroke:currentColor;stroke-width:1.25}.hero-water-line .water-bubbles circle:nth-child(3n){opacity:.5}.hero-water-line .water-bubbles circle:nth-child(3n + 2){stroke-width:1.8}.hero-water-line .water-sparkles{fill:currentColor;opacity:.76}.hero-panel{z-index:1}
  .hero-panel{position:relative;display:flex;min-height:clamp(190px,22cqw,225px);overflow:hidden;border:1px solid var(--aq-border);border-radius:var(--aq-lg);background:linear-gradient(145deg,rgba(10,42,58,.96),rgba(5,23,35,.88));box-shadow:inset 0 1px rgba(255,255,255,.03),0 15px 35px rgba(0,0,0,.14)}
  .hero-panel::after{position:absolute;right:-15%;bottom:-45%;width:70%;aspect-ratio:1;border-radius:50%;background:radial-gradient(circle,rgba(23,178,212,.1),transparent 68%);content:"pointer-events:none"}
  .status-panel,.temperature-panel{align-items:center;justify-content:space-between;gap:clamp(18px,2.6cqw,28px);padding:clamp(18px,2.7cqw,28px)}.status-panel{justify-content:flex-start;border:0;background:transparent;box-shadow:none;text-align:left}.status-panel::after{display:none}.hero-copy,.temperature-copy{position:relative;z-index:1;min-width:0}.status-panel .hero-copy{flex:1}.section-label{color:#65daf3;font-size:.72rem;font-weight:700;letter-spacing:.12em;text-transform:uppercase}
  .status-headline{display:flex;align-items:center;width:100%;height:1.92em;min-height:1.92em;max-height:1.92em;margin:9px 0 3px;overflow:hidden;overflow-wrap:anywhere;color:var(--aq-green);font-size:clamp(2rem,4.8cqw,3.6rem);font-weight:740;letter-spacing:-.035em;line-height:.96;text-shadow:0 0 20px currentColor}.status-action{color:var(--aq-green);font-size:clamp(.9rem,1.7cqw,1.08rem);font-weight:700;letter-spacing:.025em;line-height:1.15}.status-support{gap:9px;color:#c5d8e1;font-size:.86rem}.status-monitor .status-headline{font-size:clamp(1.45rem,3.3cqw,2.35rem);white-space:nowrap}.status-monitor .status-headline,.status-monitor .status-action{color:var(--aq-yellow)}.status-action_needed .status-headline{font-size:clamp(1.55rem,3.55cqw,2.65rem);white-space:nowrap}.status-action_needed .status-headline,.status-action_needed .status-action{color:var(--aq-orange)}.status-alert .status-headline,.status-alert .status-action{color:var(--aq-red)}.status-unknown .status-headline,.status-unknown .status-action{color:var(--aq-muted)}
  .status-summary{position:absolute;right:0;bottom:0;z-index:2;display:flex;width:clamp(150px,55%,270px);height:68px;flex-direction:column;justify-content:center;gap:0;padding:10px 14px;border:1px solid var(--aq-border);border-radius:var(--aq-md);background:var(--aq-surface);box-shadow:inset 0 1px rgba(255,255,255,.03),0 10px 24px rgba(0,0,0,.13)}.status-summary .status-support{position:relative;padding-top:7px}.status-summary .status-support::before{position:absolute;top:3px;left:50%;width:96%;height:1px;background:currentColor;content:"";opacity:.18;transform:translateX(-50%)}
  .status-orb{position:relative;display:grid;width:clamp(120px,16cqw,165px);aspect-ratio:1;flex:0 0 auto;place-items:center;color:var(--aq-green);--aq-status-color:var(--aq-green);margin-top:-12px}.status-indicator-svg{display:block;width:100%;height:100%;overflow:visible}.status-ring{fill:none;stroke:var(--aq-status-color);filter:url(#aquard-status-glow)}.status-ring-highlight,.status-ring-reflection{fill:none;stroke:var(--aq-gauge-highlight);stroke-linecap:round}.status-ring-highlight{stroke-width:2;opacity:.34}.status-ring-reflection{stroke-width:1.5;opacity:.2}.status-symbol{fill:none;stroke:var(--aq-status-color);stroke-width:7;stroke-linecap:round;stroke-linejoin:round;filter:url(#aquard-status-glow)}.status-symbol-fill{fill:var(--aq-status-color);filter:url(#aquard-status-glow)}.status-sparkle{fill:var(--aq-status-color);stroke:var(--aq-status-color);stroke-width:2;stroke-linecap:round;opacity:.55}.status-monitor .status-orb{--aq-status-color:var(--aq-yellow)}.status-action_needed .status-orb{--aq-status-color:var(--aq-orange)}.status-alert .status-orb{--aq-status-color:var(--aq-red)}.status-unknown .status-orb{--aq-status-color:var(--aq-muted)}.status-monitor .status-dot{background:var(--aq-yellow);box-shadow:0 0 10px color-mix(in srgb,var(--aq-yellow) 45%,transparent)}.status-action_needed .status-dot{background:var(--aq-orange);box-shadow:0 0 10px color-mix(in srgb,var(--aq-orange) 45%,transparent)}.status-alert .status-dot{background:var(--aq-red);box-shadow:0 0 10px rgba(255,100,116,.45)}.status-unknown .status-dot{background:var(--aq-muted);box-shadow:none}
  .temperature-panel{background:radial-gradient(circle at 80% 45%,rgba(13,174,220,.13),transparent 38%),linear-gradient(145deg,rgba(8,40,57,.96),rgba(5,24,37,.9))}.temperature-copy{display:flex;flex-direction:column;justify-content:center}.temperature-reading{display:flex;align-items:baseline;margin:13px 0 18px;white-space:nowrap;font-size:clamp(3.2rem,8.2cqw,6.1rem);font-weight:620;font-variant-numeric:tabular-nums;letter-spacing:-.06em;line-height:.9}.temperature-value{display:inline-flex;align-items:baseline}.temperature-decimal{font-size:.56em;font-weight:580;letter-spacing:-.035em}.temperature-unit{margin-left:.3em;color:var(--aq-muted);font-size:.27em;font-weight:500;letter-spacing:0}.metric-unit{margin-left:.22em;color:var(--aq-muted);font-size:.35em;font-weight:500;letter-spacing:0}.climate-line{flex-wrap:wrap;gap:6px 10px;padding-top:14px;border-top:1px solid rgba(89,180,205,.15);font-size:.85rem}.climate-label{color:var(--aq-blue)}.climate-value{overflow-wrap:anywhere;color:#c5dce6}
  .temperature-copy:has(.target-control) .temperature-reading{margin:8px 0}.target-control{padding-top:6px;border-top:1px solid rgba(89,180,205,.15)}.target-label{margin-bottom:3px;color:var(--aq-blue);font-size:.78rem}.target-control-row{display:grid;grid-template-columns:44px minmax(58px,1fr) 44px;align-items:center;gap:7px}.target-button{display:grid;width:44px;height:44px;padding:0;place-items:center;border:0;border-radius:13px;color:var(--aq-blue);background:transparent;cursor:pointer;transition:transform 120ms ease,opacity 160ms ease,filter 160ms ease}.target-button:hover:not(:disabled){filter:brightness(1.15)}.target-button:active:not(:disabled),.target-button.pending{transform:scale(.94);filter:brightness(1.25)}.target-button:focus-visible{outline:2px solid var(--aq-blue);outline-offset:2px}.target-button:disabled{cursor:default;opacity:.38}.target-button.pending{opacity:.72}.target-arrow-svg{display:block;width:44px;height:44px;overflow:visible}.target-button-glass{fill:rgba(7,31,45,.5);stroke:var(--aq-blue);stroke-width:1.5;filter:drop-shadow(0 0 5px rgba(24,200,243,.32))}.target-button-highlight{fill:none;stroke:var(--aq-gauge-highlight);stroke-width:1;stroke-linecap:round;opacity:.38}.target-arrow-chevron{fill:none;stroke:var(--aq-blue);stroke-width:5;stroke-linecap:round;stroke-linejoin:round;filter:drop-shadow(0 0 3px rgba(24,200,243,.55))}.target-display{display:flex;min-width:0;align-items:baseline;justify-content:center;white-space:nowrap;color:var(--aq-text);font-variant-numeric:tabular-nums}.target-number{font-size:clamp(1.55rem,3.2cqw,2.1rem);font-weight:650;letter-spacing:-.035em}.target-unit{margin-left:.3em;color:var(--aq-muted);font-size:.75rem;font-weight:500}
  .temperature-gauge{position:relative;display:grid;width:clamp(112px,16.5cqw,170px);aspect-ratio:1;align-self:center;flex:0 0 auto;place-items:center}.temperature-gauge-svg{display:block;width:100%;height:100%;overflow:visible}.temperature-gauge-track,.temperature-gauge-progress{fill:none;stroke-width:11;stroke-linecap:round}.temperature-gauge-track{stroke:var(--aq-gauge-track)}.temperature-gauge-progress{stroke:var(--aq-gauge-progress);filter:drop-shadow(0 0 6px rgba(24,200,243,.48));transition:stroke-dasharray var(--aq-motion)}.temperature-gauge-marker{fill:var(--aq-gauge-marker);filter:drop-shadow(0 0 4px var(--aq-gauge-marker));transition:cx var(--aq-motion),cy var(--aq-motion)}.temperature-gauge-target-marker{fill:none;stroke:#f0b56b;stroke-width:2;transition:cx var(--aq-motion),cy var(--aq-motion)}.temperature-gauge-target-label{fill:#f0b56b;font-size:11px;font-weight:700}.temperature-gauge-droplet{color:var(--aq-gauge-progress);transform-origin:80px 80px;animation:aq-droplet-breathe 3.6s ease-in-out infinite}.temperature-gauge-drop-edge{fill:none;stroke:var(--aq-gauge-highlight);stroke-width:1.35;stroke-opacity:.62}@keyframes aq-droplet-breathe{0%,100%{transform:translateY(0) scale(1)}50%{transform:translateY(-2.5px) scale(1.015)}}
  .temperature-gauge-svg.is-heating .temperature-gauge-progress{stroke:url(#aquard-heat-progress);filter:drop-shadow(0 0 10px rgba(255,123,0,.6))}
  .temperature-gauge-svg.is-heating .temperature-gauge-droplet{filter:drop-shadow(0 0 14px rgba(255,82,82,.7))}
  .heating-bubbles{display:none}
  .temperature-gauge-svg.is-heating .heating-bubbles{display:block}
  .heat-bubble{fill:#ff4d4d;filter:drop-shadow(0 0 4px rgba(255,77,77,.8));opacity:0;animation:aq-bubble-rise 2.4s infinite ease-in}
  .hb1{animation-delay:0s}.hb2{animation-delay:.5s}.hb3{animation-delay:1s;fill:#ff7a5c}.hb4{animation-delay:1.5s}.hb5{animation-delay:2s;fill:#ff7a5c}
  @keyframes aq-bubble-rise{0%{transform:translateY(0) scale(.6);opacity:0}20%{opacity:.95}70%{opacity:.6}100%{transform:translateY(-46px) scale(1.05);opacity:0}}
  .equipment-section{margin-top:var(--aq-gap)}.equipment-grid{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:var(--aq-gap)}.equipment-tile{display:grid;grid-template-columns:auto minmax(0,1fr) auto;align-items:center;width:100%;min-width:0;gap:14px;padding:clamp(14px,2cqw,20px);overflow:hidden;border:1px solid var(--aq-border);border-radius:var(--aq-md);color:inherit;background:linear-gradient(145deg,rgba(8,36,51,.9),rgba(6,25,38,.78));font:inherit;text-align:left;cursor:pointer;transition:transform 120ms ease,border-color 180ms ease,background 180ms ease,box-shadow 180ms ease}.equipment-tile:hover:not(:disabled){border-color:rgba(55,190,222,.4)}.equipment-tile:active:not(:disabled){transform:scale(.985)}.equipment-tile:focus-visible{outline:2px solid var(--aq-blue);outline-offset:2px}.equipment-tile:disabled{cursor:default}.equipment-tile.active{border-color:rgba(67,230,108,.3);background:linear-gradient(145deg,rgba(9,48,55,.94),rgba(6,30,40,.82));box-shadow:inset 0 0 22px rgba(67,230,108,.035)}.equipment-tile.pending{opacity:.78}.equipment-tile--split{position:relative;display:block;padding:0;cursor:default}.equipment-tile-main{display:grid;grid-template-columns:auto minmax(0,1fr) auto;align-items:center;width:100%;min-width:0;gap:14px;padding:clamp(14px,2cqw,20px);padding-right:78px;overflow:hidden;border:none;border-radius:inherit;color:inherit;background:transparent;font:inherit;text-align:left;cursor:pointer}.equipment-tile-main:disabled{cursor:default}.equipment-corner-action,.equipment-corner-countdown{position:absolute;top:10px;right:10px;z-index:2;padding:5px 10px;border-radius:999px;font-size:.72rem;font-weight:700;white-space:nowrap}.equipment-corner-action{border:1px solid var(--aq-border);background:linear-gradient(145deg,rgba(24,200,243,.18),rgba(24,200,243,.06));color:var(--aq-text);cursor:pointer}.equipment-corner-action:disabled{opacity:.45;cursor:default}.equipment-corner-countdown{border:1px solid rgba(24,200,243,.4);background:rgba(24,200,243,.1);color:var(--aq-blue);font-variant-numeric:tabular-nums}.equipment-tile--split .status-dot{position:absolute;top:50%;right:clamp(14px,2cqw,20px);transform:translateY(-50%);z-index:1}
  .equipment-icon{display:grid;width:clamp(48px,5.8cqw,62px);aspect-ratio:1;place-items:center;border:2px solid var(--aq-blue);border-radius:50%;color:var(--aq-blue);box-shadow:0 0 18px rgba(25,199,242,.13),inset 0 0 14px rgba(25,199,242,.08)}.equipment-icon ha-icon{width:27px;height:27px;--mdc-icon-size:27px}.equipment-copy{min-width:0}.equipment-name{color:#dcebf1;font-size:clamp(.92rem,1.7cqw,1.08rem);font-weight:600}.equipment-value{margin-top:4px;overflow-wrap:anywhere;color:#65daf3;font-size:.86rem}.available .equipment-value{color:var(--aq-green)}.pending .status-dot{border:2px solid rgba(101,218,243,.28);border-top-color:var(--aq-blue);background:transparent;box-shadow:none;animation:aq-spin .7s linear infinite}@keyframes aq-spin{to{transform:rotate(360deg)}}
  .measurement-section{margin-top:var(--aq-gap);overflow:hidden;border:1px solid var(--aq-border);border-radius:var(--aq-lg);background:rgba(5,24,36,.72)}.metric-grid{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:1px;background:rgba(62,176,207,.16)}.metric-tile{position:relative;min-width:0;padding:clamp(17px,2.3cqw,24px);overflow:hidden;background:linear-gradient(145deg,rgba(8,36,51,.94),rgba(6,25,38,.88));--range-intensity:0;--range-opacity:.42;--current-position:50%}.metric-corner-badge{position:absolute;top:10px;right:10px;z-index:2;display:flex;align-items:center;gap:4px;padding:4px 10px;border-radius:999px;border:1px solid rgba(240,181,107,.4);background:rgba(240,181,107,.12);color:#f0b56b;font-size:1.05rem;font-weight:750;white-space:nowrap}.metric-corner-badge ha-icon{--mdc-icon-size:16px;width:16px;height:16px}.metric-heading,.metric-label,.metric-reading-row,.metric-footer{display:flex;align-items:center;min-width:0}.metric-label{gap:9px}.metric-icon{width:23px;height:23px;flex:0 0 auto;color:var(--aq-blue);--mdc-icon-size:23px}.metric-name{color:#bcd4df;font-size:.9rem;font-weight:620;letter-spacing:.02em}.metric-reading-row{justify-content:space-between;gap:10px;margin-top:14px}.metric-reading{min-width:0;overflow-wrap:anywhere;font-size:clamp(1.85rem,3.8cqw,2.7rem);font-weight:590;letter-spacing:-.035em}.metric-quality-mark{display:grid;width:31px;height:31px;flex:0 0 auto;place-items:center;border:2px solid var(--aq-green);border-radius:50%;color:var(--aq-green);box-shadow:0 0 12px rgba(67,230,108,.14)}.metric-quality-mark ha-icon{width:18px;height:18px;--mdc-icon-size:18px}.quality-monitor .metric-quality-mark{border-color:var(--aq-yellow);color:var(--aq-yellow)}.quality-action_needed .metric-quality-mark{border-color:var(--aq-orange);color:var(--aq-orange)}.quality-alert .metric-quality-mark{border-color:var(--aq-red);color:var(--aq-red)}.unavailable .metric-quality-mark,.not-configured .metric-quality-mark{border-color:var(--aq-muted);color:var(--aq-muted);opacity:.55}.metric-meter{position:relative;height:8px;margin-top:16px;overflow:visible;border-radius:999px;color:var(--aq-muted);background:rgba(37,68,82,.5);isolation:isolate}.metric-meter::before{position:absolute;inset:0;z-index:0;border-radius:inherit;background:var(--meter-color,currentColor);opacity:var(--range-opacity,.4);content:"";transition:background 300ms ease,opacity 300ms ease}.range-ideal .metric-meter{color:var(--aq-green);--range-opacity:.82}.range-neutral .metric-meter{color:var(--aq-muted);--range-opacity:.28}.metric-value-marker{position:absolute;top:50%;left:clamp(4px,var(--current-position),calc(100% - 4px));z-index:2;width:2px;height:16px;border-radius:2px;background:#f4fbff;box-shadow:0 0 5px rgba(255,255,255,.8);transform:translate(-50%,-50%);transition:left var(--aq-motion)}.range-neutral .metric-value-marker{display:none}.metric-footer{justify-content:space-between;gap:8px;margin-top:10px}.metric-quality{min-width:0;color:var(--aq-muted);font-size:.7rem;overflow-wrap:anywhere}.metric-state{min-width:0;flex:0 0 auto;gap:5px}.quality-excellent .metric-quality{color:var(--aq-green)}.quality-monitor .metric-quality{color:var(--aq-yellow)}.quality-action_needed .metric-quality{color:var(--aq-orange)}.quality-alert .metric-quality{color:var(--aq-red)}
  .hero-grid--focused{grid-template-columns:minmax(0,1fr)}.hero-grid--focused .hero-panel{max-width:none}.status-panel--compact,.temperature-panel--compact{position:relative;z-index:1;display:flex;min-width:0;min-height:112px;align-items:center;gap:16px;padding:16px 18px;overflow:hidden;border:1px solid var(--aq-border);border-radius:var(--aq-lg);background:linear-gradient(145deg,rgba(8,40,57,.94),rgba(5,24,37,.86))}.status-panel--compact{padding-right:clamp(140px,42%,220px)}.status-panel--compact .status-orb{width:72px}.status-panel--compact .status-headline{height:auto;min-height:0;max-height:none;margin:0;font-size:clamp(1.35rem,3.4cqw,2.1rem)}.status-score{margin-top:6px;color:var(--aq-muted);font-size:.78rem}.status-summary--compact{height:54px}.actions-standalone{position:relative;width:100%;height:auto;min-height:64px}.temperature-panel--compact{justify-content:space-between}.temperature-panel--compact .temperature-reading{margin:7px 0 0;font-size:clamp(2.4rem,6cqw,4rem)}.temperature-panel--compact .target-control{margin-top:8px}.temperature-panel--compact .target-label{display:none}.measurement-section--compact .metric-grid{grid-template-columns:repeat(auto-fit,minmax(120px,1fr))}.measurement-section--compact .metric-tile{padding:14px}.measurement-section--compact .metric-reading-row{margin-top:8px}.measurement-section--compact .metric-reading{font-size:1.55rem}.measurement-section--compact .metric-meter,.measurement-section--compact .metric-footer{display:none}.equipment-section--compact .equipment-grid{grid-template-columns:repeat(auto-fit,minmax(112px,1fr));gap:8px}.equipment-section--compact .equipment-tile{min-height:48px;gap:9px;padding:8px 10px}.equipment-section--compact .equipment-icon{width:40px}.equipment-section--compact .equipment-icon ha-icon{width:22px;height:22px;--mdc-icon-size:22px}.equipment-section--compact .equipment-value{margin-top:1px;font-size:.75rem}.aquard-header--compact{margin-bottom:12px}.aquard-header--compact .brand-name{font-size:1.35rem}
  .equipment-grid{grid-template-columns:repeat(auto-fit,minmax(min(180px,100%),1fr))}.metric-grid{grid-template-columns:repeat(auto-fit,minmax(min(180px,100%),1fr))}.equipment-grid[data-count="1"],.metric-grid[data-count="1"]{grid-template-columns:minmax(0,min(100%,360px))}
  @media(prefers-reduced-motion:reduce){.equipment-tile,.target-button,.temperature-gauge-progress,.temperature-gauge-marker{transition-duration:.01ms}.pending .status-dot{animation-duration:1.4s}.temperature-gauge-droplet{animation:none}.heat-bubble{animation:none;opacity:.5}}
  @container(max-width:760px){
    .hero-panel{min-height:205px}
    .metric-grid{grid-template-columns:repeat(2,minmax(0,1fr))}
    .status-headline{white-space:normal;height:auto;min-height:0;max-height:none;line-height:1.1}
    .hero-panel .status-headline{font-size:clamp(1.4rem,4cqw,2.2rem)}
    .hero-panel .temperature-reading{font-size:clamp(2.2rem,6cqw,3.5rem)}
  }
  @container(max-width:560px){
    ha-card{border-radius:22px}
    .header-availability{display:none}
    .hero-panel.status-panel,.hero-panel.temperature-panel{min-height:195px;padding:20px}
    .hero-panel .status-orb{width:92px}
    .temperature-gauge{width:116px}
    .equipment-section:not(.equipment-section--compact) .equipment-grid{grid-template-columns:minmax(0,1fr)}
  }
  @container(max-width:390px){
    .hero-panel.status-panel,.hero-panel.temperature-panel{min-height:0}
    .hero-panel .status-orb{width:82px}
    .temperature-gauge{width:102px}
    .hero-panel .status-headline{font-size:1.35rem}
    .hero-panel .temperature-reading{font-size:2.1rem}
    .measurement-section:not(.measurement-section--compact) .metric-grid{grid-template-columns:repeat(2,minmax(0,1fr))}
    .metric-state-text{display:none}
  }
  .wp-panel{margin-top:var(--aq-gap);padding:clamp(14px,2cqw,18px);border:1px solid var(--aq-border);border-radius:var(--aq-lg);background:linear-gradient(145deg,rgba(8,36,51,.9),rgba(6,25,38,.78))}
  .wp-panel-header{display:flex;align-items:center;gap:9px;margin-bottom:10px}
  .wp-panel-icon{width:22px;height:22px;--mdc-icon-size:22px;color:var(--aq-blue)}
  .wp-panel-title{flex:1;min-width:0;color:#dcebf1;font-size:.95rem;font-weight:650;letter-spacing:.01em}
  .wp-status-pill{padding:3px 10px;border-radius:999px;border:1px solid var(--aq-border);color:var(--aq-muted);font-size:.7rem;font-weight:650;letter-spacing:.03em;text-transform:uppercase;white-space:nowrap}
  .wp-status-pill.available{border-color:rgba(67,230,108,.4);color:var(--aq-green)}
  .wp-status-pill.unavailable{border-color:rgba(240,181,107,.4);color:var(--aq-amber,var(--aq-orange))}
  .wp-panel-body{display:flex;flex-direction:column;gap:8px}
  .wp-row{display:flex;align-items:center;justify-content:space-between;gap:12px;font-size:.86rem}
  .wp-row-label{color:var(--aq-muted)}
  .wp-row-value{overflow-wrap:anywhere;color:#c5dce6;text-align:right}
  .wp-countdown{font-variant-numeric:tabular-nums;color:var(--aq-blue);font-size:1.3rem;font-weight:650;letter-spacing:-.02em}
  .wp-duration-row{align-items:center}
  .wp-duration-control{display:flex;align-items:center;gap:10px}
  .wp-duration-value{min-width:64px;font-variant-numeric:tabular-nums;font-weight:650}
  .wp-stepper{display:grid;width:32px;height:32px;place-items:center;border:1px solid var(--aq-border);border-radius:10px;color:var(--aq-blue);background:transparent;font-size:1.1rem;font-weight:700;cursor:pointer;transition:filter 160ms ease}
  .wp-stepper:hover{filter:brightness(1.2)}
  .wp-stepper:active{transform:scale(.94)}
  .wp-button{width:100%;padding:11px;border:1px solid var(--aq-border);border-radius:12px;color:var(--aq-text);background:linear-gradient(145deg,rgba(24,200,243,.16),rgba(24,200,243,.05));font:inherit;font-weight:650;letter-spacing:.02em;cursor:pointer;transition:filter 160ms ease,transform 120ms ease}
  .wp-button:hover{filter:brightness(1.12)}
  .wp-button:active{transform:scale(.985)}
  .wp-button-stop{border-color:rgba(255,100,116,.4);background:linear-gradient(145deg,rgba(255,100,116,.18),rgba(255,100,116,.05));color:#ffb2ba}
  .wp-button:disabled{cursor:default;opacity:.45}
  .wp-panel-header:has(.wp-button-inline),.wp-panel-header:has(.wp-countdown-inline){flex-wrap:nowrap}
  .wp-button-inline{flex:0 0 auto;width:auto;padding:7px 14px;font-size:.82rem}
  .wp-countdown-inline{flex:0 0 auto;font-size:1rem}
  .wp-row-icon{--mdc-icon-size:16px;margin-right:6px;vertical-align:-3px;color:var(--aq-blue)}
  .wp-dose-value{font-variant-numeric:tabular-nums;font-weight:650;color:#f0b56b}
  .timers-grid { margin-top: var(--aq-gap); display: grid; gap: var(--aq-gap); grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); }
  .timers-grid .wp-panel { margin-top: 0; height: 100%; display: flex; flex-direction: column; }
  .timers-grid .wp-panel-body { flex: 1; justify-content: center; }
`;

const AQUARD_PROFILES = Object.freeze([
  { value: "spa", label: "Spa" },
]);

const DEVICE_FIELDS = Object.freeze([
  { key: "ph", label: "pH-Sensor", group: "Wasserueberwachung", domains: ["sensor"] },
  { key: "orp", label: "ORP-Sensor", group: "Wasserueberwachung", domains: ["sensor"] },
  { key: "tds", label: "TDS-Sensor", group: "Wasserueberwachung", domains: ["sensor"] },
  { key: "ec", label: "EC-Sensor", group: "Wasserueberwachung", domains: ["sensor"] },
  { key: "climate", label: "Whirlpool-Klimaeinheit", group: "Temperaturregelung", domains: ["climate"], description: "Stellt die Zieltemperatur-Steuerung bereit." },
  { key: "time_to_ready", label: "Restzeit bis Zieltemp.", group: "Temperaturregelung", domains: ["sensor"] },
  { key: "auto_mode", label: "Automatik-Modus (Input Boolean)", group: "Modus-Steuerung", domains: ["input_boolean", "switch"] },
  { key: "manual_mode", label: "Manuell-Modus (Input Boolean)", group: "Modus-Steuerung", domains: ["input_boolean", "switch"] },
  { key: "water_temperature", label: "Wassertemperatur-Sensor", group: "Temperaturregelung", domains: ["sensor"] },
  { key: "power", label: "Ein/Aus", group: "Whirlpool-Geraete", domains: ["switch"] },
  { key: "heater", label: "Heizung", group: "Whirlpool-Geraete", domains: ["climate", "switch"] },
  { key: "filter", label: "Filter", group: "Whirlpool-Geraete", domains: ["switch"] },
  { key: "bubbles", label: "Sprudel", group: "Whirlpool-Geraete", domains: ["switch", "select"] },
  { key: "calendar", label: "Kalender (Termine)", group: "Vorheizen & Timer", domains: ["calendar"], description: "Fuer die Anzeige des naechsten Termins." },
  { key: "preheat_needed", label: "Vorheizen noetig (Sensor)", group: "Vorheizen & Timer", domains: ["binary_sensor"] },
  { key: "preheat_start_time", label: "Geplante Vorheiz-Startzeit", group: "Vorheizen & Timer", domains: ["sensor"] },
  { key: "heat_timer", label: "Timer: Heizen auf Zeit", group: "Vorheizen & Timer", domains: ["timer"] },
  { key: "heat_duration", label: "Dauer: Heizen auf Zeit", group: "Vorheizen & Timer", domains: ["input_number"] },
  { key: "heat_start", label: "Skript: Heizen starten", group: "Vorheizen & Timer", domains: ["script"] },
  { key: "heat_stop", label: "Skript: Heizen stoppen", group: "Vorheizen & Timer", domains: ["script"] },
  { key: "filter_timer", label: "Timer: Schnellfiltern", group: "Vorheizen & Timer", domains: ["timer"] },
  { key: "filter_quick_start", label: "Skript: Schnellfiltern starten", group: "Vorheizen & Timer", domains: ["script"] },
]);

const ICON_FIELDS = Object.freeze([
  { key: "power", label: "Icon: Ein/Aus", fallback: "mdi:power" },
  { key: "filter", label: "Icon: Filter", fallback: "mdi:air-filter" },
  { key: "heater", label: "Icon: Heizung", fallback: "mdi:radiator" },
  { key: "bubbles", label: "Icon: Sprudel", fallback: "mdi:chart-bubble" },
  { key: "ph", label: "Icon: pH", fallback: "mdi:water-outline" },
  { key: "orp", label: "Icon: ORP", fallback: "mdi:shield-check-outline" },
  { key: "ec", label: "Icon: EC", fallback: "mdi:pulse" },
  { key: "tds", label: "Icon: TDS", fallback: "mdi:dots-circle" },
]);

const LABEL_FIELDS = Object.freeze([
  { key: "waterTemperature", label: "Ueberschrift: Wassertemperatur", fallback: "Wassertemperatur" },
  { key: "power", label: "Ueberschrift: Ein/Aus-Kachel", fallback: "Ein/Aus" },
  { key: "filter", label: "Ueberschrift: Filter-Kachel", fallback: "Filter" },
  { key: "heater", label: "Ueberschrift: Heizungs-Kachel", fallback: "Heizung" },
  { key: "bubbles", label: "Ueberschrift: Sprudel-Kachel", fallback: "Sprudel" },
  { key: "preheatTitle", label: "Ueberschrift: Kalender-Vorheizen", fallback: "Kalender-Vorheizen" },
  { key: "timedHeatingTitle", label: "Ueberschrift: Heizen auf Zeit", fallback: "Heizen auf Zeit" },
]);

const COMPONENT_OPTIONS = Object.freeze([
  { key: "water_status", label: "Water status", description: "Overall water condition and score." },
  { key: "temperature", label: "Temperature", description: "Current and target temperature." },
  { key: "actions", label: "Actions", description: "Water-care guidance and warnings." },
  { key: "measurements", label: "Measurements", description: "pH, ORP, EC, and TDS readings." },
  { key: "controls", label: "Controls", description: "Spa equipment controls." },
  { key: "details", label: "Details", description: "Card title and availability." },
]);

const MODE_OPTIONS = Object.freeze([
  { value: "full", label: "Full" },
  { value: "compact", label: "Compact" },
  { value: "hidden", label: "Hidden" },
]);

function updateConfigProperty(config, path, value) {
  const nextConfig = { ...(config ?? {}) };
  if (path.length === 1) {
    setOrDelete(nextConfig, path[0], value);
    return nextConfig;
  }

  const [parent, property] = path;
  nextConfig[parent] = { ...(config?.[parent] ?? {}) };
  setOrDelete(nextConfig[parent], property, value);
  return nextConfig;
}

function dispatchConfigChanged(element, config) {
  element.dispatchEvent(new CustomEvent("config-changed", {
    detail: { config },
    bubbles: true,
    composed: true,
  }));
}

function hasMeaningfulEntities(config) {
  return Boolean(config?.entities && Object.values(config.entities).some((value) => typeof value === "string" && value.trim()));
}

function setOrDelete(target, property, value) {
  if (value === "" || value === undefined || value === null) delete target[property];
  else target[property] = value;
}


const LAYOUT_PRESETS = Object.freeze({
  dashboard: Object.freeze({ ...DEFAULT_COMPONENT_MODES }),
  compact: Object.freeze({
    water_status: "compact",
    temperature: "compact",
    actions: "hidden",
    measurements: "compact",
    controls: "hidden",
    details: "hidden",
  }),
});

function getEffectiveComponents(config) {
  return Object.fromEntries(COMPONENT_IDS.map((id) => [id, config?.components?.[id] ?? DEFAULT_COMPONENT_MODES[id]]));
}

function deriveLayoutPreset(config) {
  const modes = getEffectiveComponents(config);
  for (const [preset, mapping] of Object.entries(LAYOUT_PRESETS)) {
    if (COMPONENT_IDS.every((id) => modes[id] === mapping[id])) return preset;
  }
  return "custom";
}

function applyLayoutPreset(config, preset) {
  if (preset === "custom" || !LAYOUT_PRESETS[preset]) return config;
  return { ...config, components: { ...(config?.components ?? {}), ...LAYOUT_PRESETS[preset] } };
}


const selectMarkup = (options) => options.map(({ value, label }) => `<mwc-list-item value="${value}">${label}</mwc-list-item>`).join("");

class AquardCardEditor extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
  }

  setConfig(config) {
    this._config = config && typeof config === "object" && !Array.isArray(config) ? config : {};
    this._render();
  }

  set hass(hass) {
    this._hass = hass;
    this._syncHass();
  }

  _render() {
    if (!this.shadowRoot || !this._config) return;
    const profile = this._config.profile ?? "spa";
    const groups = [...new Set(DEVICE_FIELDS.map((field) => field.group))];
    this.shadowRoot.innerHTML = `
      <style>
        :host{display:block;color:var(--primary-text-color)}*{box-sizing:border-box}.notice{display:flex;gap:10px;margin:0 0 16px;padding:12px 14px;border-radius:10px;background:var(--secondary-background-color);line-height:1.4}.notice ha-icon{flex:0 0 auto;color:var(--primary-color)}details{border-top:1px solid var(--divider-color)}details:first-of-type{border-top:0}summary{padding:18px 2px;font-size:16px;font-weight:500;cursor:pointer;list-style-position:inside}.section-body{padding:0 2px 18px}.section-copy,.field-copy{margin:-4px 0 16px;color:var(--secondary-text-color);font-size:13px;line-height:1.4}.group{margin-top:20px}.group:first-child{margin-top:0}.group-title{margin:0 0 12px;font-size:14px;font-weight:600}ha-select,ha-entity-picker,ha-textfield{display:block;width:100%;margin-bottom:16px}.switch-row{display:flex;align-items:flex-start;justify-content:space-between;gap:20px}.switch-label{font-weight:500}.switch-description{margin:4px 0 0}.sensor-information-toggle{flex:0 0 auto}.error{color:var(--error-color)}
      </style>
      ${!hasMeaningfulEntities(this._config) ? `<div class="notice" role="status"><ha-icon icon="mdi:information-outline"></ha-icon><span>Waehle unten mindestens eine Whirlpool-Entitaet aus. Du kannst jetzt speichern und die Einrichtung spaeter fortsetzen.</span></div>` : ""}
      <details open><summary>Profil</summary><div class="section-body"><p class="section-copy">Waehle das Wasserprofil, das diese Karte darstellt.</p><ha-select class="profile-select" label="Profil">${selectMarkup(AQUARD_PROFILES)}</ha-select>${AQUARD_PROFILES.some((item) => item.value === profile) ? "" : `<p class="field-copy error" role="alert">Das eingestellte Profil wird nicht unterstuetzt. Waehle "Spa", um fortzufahren.</p>`}</div></details>
      <details open><summary>Entitaeten</summary><div class="section-body"><p class="section-copy">Alle Entitaeten sind optional. Fehlende oder nicht verfuegbare Entity-IDs bleiben erhalten, damit sie spaeter wieder verbunden werden koennen.</p>${groups.map((group) => `<section class="group"><h4 class="group-title">${group}</h4><div data-device-group="${group}"></div></section>`).join("")}</div></details>
      <details><summary>Icons</summary><div class="section-body"><p class="section-copy">Icons der Kacheln individuell anpassen.</p><div data-icon-fields></div></div></details>
      <details><summary>Ueberschriften</summary><div class="section-body"><p class="section-copy">Beschriftungen der Kacheln und Bereiche individuell anpassen.</p><div data-label-fields></div></div></details>
      <details open><summary>Optionen</summary><div class="section-body"><div class="switch-row"><div><div class="switch-label">Sensordaten anzeigen</div><div class="field-copy switch-description" id="sensor-information-description">Zeigt die pH-, ORP-, EC- und TDS-Kacheln an, wenn konfiguriert.</div></div><ha-switch class="sensor-information-toggle" aria-label="Sensordaten anzeigen" aria-describedby="sensor-information-description"></ha-switch></div></div></details>
      <details><summary>Erweitert</summary><div class="section-body"><p class="section-copy">Weitere Karteneinstellungen.</p><ha-textfield class="name-field" label="Kartenname"></ha-textfield></div></details>`;

    const profileSelect = this.shadowRoot.querySelector(".profile-select");
    profileSelect.value = profile;
    profileSelect.addEventListener("selected", (event) => this._change(["profile"], event.target.value));

    for (const field of DEVICE_FIELDS) {
      const picker = document.createElement("ha-entity-picker");
      picker.label = field.label;
      picker.value = this._config.entities?.[field.key] ?? "";
      picker.hass = this._hass;
      picker.includeDomains = field.domains;
      picker.allowCustomEntity = true;
      picker.helper = field.description ?? "Optional";
      picker.addEventListener("value-changed", (event) => this._change(["entities", field.key], event.detail?.value));
      this.shadowRoot.querySelector(`[data-device-group="${field.group}"]`).append(picker);
    }

    const iconFieldsContainer = this.shadowRoot.querySelector("[data-icon-fields]");
    for (const field of ICON_FIELDS) {
      const picker = document.createElement("ha-icon-picker");
      picker.label = field.label;
      picker.value = this._config.icons?.[field.key] ?? "";
      picker.hass = this._hass;
      picker.placeholder = field.fallback;
      picker.addEventListener("value-changed", (event) => this._change(["icons", field.key], event.detail?.value));
      iconFieldsContainer.append(picker);
    }

    const labelFieldsContainer = this.shadowRoot.querySelector("[data-label-fields]");
    for (const field of LABEL_FIELDS) {
      const textfield = document.createElement("ha-textfield");
      textfield.label = field.label;
      textfield.placeholder = field.fallback;
      textfield.value = this._config.labels?.[field.key] ?? "";
      textfield.addEventListener("input", (event) => this._change(["labels", field.key], event.target.value));
      labelFieldsContainer.append(textfield);
    }

    const sensorInformationToggle = this.shadowRoot.querySelector(".sensor-information-toggle");
    sensorInformationToggle.checked = this._config.show_sensor_information !== false;
    sensorInformationToggle.addEventListener("change", (event) => this._change(["show_sensor_information"], event.target.checked));

    const name = this.shadowRoot.querySelector(".name-field");
    name.value = this._config.name ?? "";
    name.addEventListener("input", (event) => this._change(["name"], event.target.value));
  }

  _syncHass() {
    if (!this.shadowRoot) return;
    for (const picker of this.shadowRoot.querySelectorAll("ha-entity-picker, ha-icon-picker")) picker.hass = this._hass;
  }

  _change(path, value) {
    const config = updateConfigProperty(this._config, path, value);
    this._config = config;
    dispatchConfigChanged(this, config);
  }
}

if (!customElements.get("aquard-card-whirlpool-editor")) customElements.define("aquard-card-whirlpool-editor", AquardCardEditor);


const METRICS = [
  ["ph", "pH", "mdi:water-outline"],
  ["orp", "ORP", "mdi:shield-check-outline"],
  ["ec", "EC", "mdi:pulse"],
  ["tds", "TDS", "mdi:dots-circle"],
];

const METRIC_ICON_FALLBACKS = Object.freeze({
  ph: "mdi:water-outline",
  orp: "mdi:shield-check-outline",
  ec: "mdi:pulse",
  tds: "mdi:dots-circle",
});

const EQUIPMENT_ICONS = Object.freeze({
  power: "mdi:power",
  filter: "mdi:air-filter",
  heater: "mdi:radiator",
  bubbles: "mdi:chart-bubble",
});

const UI_TEXT = Object.freeze({
  brand: "Aquard",
  dashboard: "Wasserueberwachung",
  waterTemperature: "Wassertemperatur",
  climateTarget: "Zieltemperatur",
  equipmentStatus: "Geraetestatus",
  waterQualityMeasurements: "Wasserqualitaet",
  quality: "Qualitaet",
  rangeLow: "Unter Idealbereich",
  rangeIdeal: "Im Idealbereich",
  rangeHigh: "Ueber Idealbereich",
  rangeNeutral: "Bereich nicht verfuegbar",
  available: "Sensor verfuegbar",
  power: "Ein/Aus",
  filter: "Filter",
  heater: "Heizung",
  bubbles: "Sprudel",
  preheatTitle: "Kalender-Vorheizen",
  preheatNeededYes: "Vorheizen laeuft",
  preheatNeededNo: "Kein Vorheizen noetig",
  preheatNeededUnknown: "Status unbekannt",
  preheatNextAppointment: "Naechster Termin",
  preheatNoAppointment: "Kein Termin geplant",
  preheatStartTime: "Geplanter Heizstart",
  preheatStartTimeUnknown: "Wird berechnet",
  timedHeatingTitle: "Heizen auf Zeit",
  timedHeatingDuration: "Dauer",
  timedHeatingMinutes: "Min",
  timedHeatingStart: "Heizen starten",
  timedHeatingStop: "Heizen stoppen",
  timedHeatingRemaining: "Verbleibend",
  timedHeatingActive: "Heizung laeuft",
  timedHeatingIdle: "Bereit",
  quickFilterTitle: "Filtern",
  quickFilterStart: "15 Min",
  dosingTitle: "Dosierung",
  dosingPhPlus: "pH-Plus",
  dosingPhMinus: "pH-Minus",
  dosingChlor: "Chlor",
  dosingNoneNeeded: "Kein Nachfuellen noetig",
  dosingUnavailable: "Werte nicht verfuegbar",
});

const WATER_STATUS_TEXT = Object.freeze({
  excellent: "AUSGEZEICHNET",
  monitor: "BEOBACHTEN",
  action_needed: "HANDLUNG NOETIG",
  alert: "ALARM",
  unknown: "UNBEKANNT",
});

const WATER_ACTION_TEXT = Object.freeze({
  excellent: "KEINE AKTION NOETIG",
  monitor: "IM BLICK BEHALTEN",
  action_needed: "AKTION ERFORDERLICH",
  alert: "SOFORT HANDELN",
  unknown: "STATUS NICHT VERFUEGBAR",
});

const WATER_MESSAGE_TEXT = Object.freeze({
  enjoy_your_spa: "Viel Spass im Whirlpool.",
  keep_monitoring: "Weiter beobachten.",
  ph_slightly_below_ideal: "pH liegt leicht unter dem Idealwert.",
  ph_slightly_above_ideal: "pH liegt leicht ueber dem Idealwert.",
  sanitizer_performance_declining: "Die Desinfektionsleistung laesst nach.",
  tds_gradually_increasing: "TDS steigt allmaehlich an.",
  raise_ph: "pH-Wert erhoehen.",
  lower_ph: "pH-Wert senken.",
  raise_ph_before_use: "pH-Wert vor Nutzung erhoehen.",
  lower_ph_before_use: "pH-Wert vor Nutzung senken.",
  restore_sanitizer: "Desinfektionsmittel auffuellen.",
  verify_sanitizer_before_use: "Desinfektion vor Nutzung pruefen.",
  correct_ph_before_use: "pH-Wert vor Nutzung korrigieren.",
  replace_water_soon: "Wasser bald wechseln.",
  replace_water_before_use: "Wasser vor Nutzung pruefen oder wechseln.",
  sensor_data_unavailable: "Sensordaten nicht verfuegbar.",
  maintenance_due: "Wartung sollte bald erfolgen.",
  check_water_before_use: "Wasser vor Nutzung pruefen.",
});

const TARGET_DEBOUNCE_MS = 400;
const TARGET_CONFIRMATION_TIMEOUT_MS = 5000;

const WATER_LINE_DECORATION = `
  <svg class="hero-water-line" viewBox="0 0 1200 260" preserveAspectRatio="none" aria-hidden="true">
    <path class="water-ribbon" d="M-30 178 C105 124 190 238 342 206 C506 171 570 126 718 169 C884 217 1010 222 1230 135 L1230 181 C1028 243 885 239 714 197 C555 158 467 220 325 237 C174 254 72 181 -30 209Z"/>
    <path class="water-ribbon water-ribbon-secondary" d="M-30 207 C118 152 240 247 394 217 C548 187 653 151 809 194 C955 234 1083 211 1230 162 L1230 194 C1070 238 934 254 793 220 C646 184 548 224 391 245 C226 267 101 196 -30 231Z"/>
    <path class="water-wave water-wave-primary" d="M-30 175 C108 119 194 232 344 201 C497 169 580 119 727 164 C887 214 1014 218 1230 132"/>
    <path class="water-wave" d="M-30 190 C116 139 218 239 371 208 C531 176 620 137 775 181 C925 224 1051 214 1230 151"/>
    <path class="water-wave water-line-secondary" d="M-30 212 C128 160 251 248 413 219 C566 191 670 158 824 198 C973 237 1095 211 1230 172"/>
    <path class="water-filament" d="M-30 224 C152 184 248 258 432 230 S723 190 888 223 S1087 224 1230 192"/>
    <g class="water-bubbles">
      <circle cx="28" cy="158" r="3"/><circle cx="42" cy="174" r="7"/><circle cx="67" cy="145" r="2"/>
      <circle cx="88" cy="190" r="4"/><circle cx="112" cy="161" r="6"/><circle cx="139" cy="184" r="2.5"/>
      <circle cx="171" cy="199" r="5"/><circle cx="205" cy="171" r="3"/><circle cx="238" cy="211" r="2"/>
      <circle cx="302" cy="181" r="2.5"/><circle cx="338" cy="160" r="5"/><circle cx="374" cy="189" r="3"/>
      <circle cx="421" cy="215" r="2"/><circle cx="475" cy="178" r="4"/><circle cx="532" cy="153" r="2.5"/>
      <circle cx="594" cy="182" r="5"/><circle cx="642" cy="143" r="3"/><circle cx="681" cy="166" r="2"/>
      <circle cx="733" cy="139" r="6"/><circle cx="762" cy="177" r="3"/><circle cx="806" cy="157" r="4"/>
      <circle cx="852" cy="192" r="2.5"/><circle cx="899" cy="169" r="5"/><circle cx="944" cy="201" r="3"/>
      <circle cx="995" cy="176" r="2"/><circle cx="1038" cy="190" r="6"/><circle cx="1081" cy="159" r="3"/>
      <circle cx="1124" cy="181" r="4"/><circle cx="1165" cy="145" r="2.5"/><circle cx="1192" cy="173" r="5"/>
    </g>
    <g class="water-sparkles">
      <circle cx="54" cy="205" r="1.5"/><circle cx="150" cy="151" r="1"/><circle cx="267" cy="191" r="1.5"/>
      <circle cx="391" cy="169" r="1"/><circle cx="514" cy="204" r="1.5"/><circle cx="706" cy="191" r="1"/>
      <circle cx="838" cy="145" r="1.5"/><circle cx="972" cy="183" r="1"/><circle cx="1108" cy="211" r="1.5"/>
    </g>
  </svg>`;

export class AquardCard extends HTMLElement {
  static getConfigElement() {
    return document.createElement("aquard-card-whirlpool-editor");
  }

  static getStubConfig() {
    return {
      profile: "spa",
      entities: {
        ph: "sensor.wasserqualitat_sensor_ph",
        climate: "climate.layzspa_wifi_controller_layzspa_temperature_control",
        heater: "switch.layzspa_wifi_controller_layzspa_heat_regulation",
        time_to_ready: "sensor.whirlpool_vorheizstunden",
        orp: "sensor.whirlpool_orp_geglattet",
        filter: "switch.layzspa_wifi_controller_layzspa_pump",
        bubbles: "switch.blubbern",
        calendar: "calendar.whirlpool",
        preheat_needed: "binary_sensor.whirlpool_vorheizen_noetig",
        heat_timer: "timer.whirlpool_heizen_auf_zeit_timer",
        heat_duration: "input_number.whirlpool_heizen_auf_zeit_dauer",
        heat_start: "script.whirlpool_heizen_starten",
        heat_stop: "script.whirlpool_heizen_stoppen",
        filter_timer: "timer.whirlpool_filtern_timer",
        filter_quick_start: "script.whirlpool_filtern_15_min",
        preheat_start_time: "sensor.whirlpool_vorheiz_startzeit"
      },
      components: {
        water_status: "full",
        temperature: "full",
        actions: "full",
        measurements: "full",
        controls: "full",
        details: "full",
      },
    };
  }

  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this._pendingControls = new Set();
    this._pendingTarget = null;
    this._targetDebounceTimer = null;
    this._targetConfirmationTimer = null;
    this._countdownTicker = null;
    this._pendingHeatAction = false;
    this._renderGeneration = 0;
    this._minHeightReleaseTimer = null;
  }

  disconnectedCallback() {
    clearTimeout(this._targetDebounceTimer);
    this._clearTargetConfirmationTimer();
    clearInterval(this._countdownTicker);
    this._countdownTicker = null;
    clearTimeout(this._minHeightReleaseTimer);
    this._minHeightReleaseTimer = null;
  }

  setConfig(config) {
    this._config = normalizeAquardConfig(config);
    this._render();
  }

  set hass(hass) {
    this._hass = hass;
    this._reconcilePendingTarget();
    const signature = this._computeSignature();
    if (signature !== null && signature === this._lastSignature) return;
    this._lastSignature = signature;
    this._render();
  }

  _computeSignature() {
    const entities = this._config?.entities;
    if (!entities || !this._hass?.states) return null;
    let sig = "";
    for (const entityId of Object.values(entities)) {
      if (!entityId) continue;
      const stateObj = this._hass.states[entityId];
      if (!stateObj) { sig += `${entityId}:__missing__|`; continue; }
      sig += `${entityId}:${stateObj.state}:${stateObj.attributes?.temperature ?? ""}:${stateObj.attributes?.current_temperature ?? ""}:${stateObj.attributes?.finishes_at ?? ""}:${stateObj.attributes?.start_time ?? ""}|`;
    }
    return sig;
  }

  getCardSize() {
    if (!this._config) return 1;
    const weights = { water_status: [2, 1], temperature: [2, 1], actions: [1, 1], measurements: [2, 1], controls: [1, 1], details: [1, 1] };
    return Math.max(1, Object.entries(weights).reduce((size, [componentId, [full, compact]]) => {
      const mode = getComponentMode(this._config, componentId);
      return size + (mode === "full" ? full : mode === "compact" ? compact : 0);
    }, 0));
  }

  getGridOptions() {
    return { columns: 12, min_columns: 6 };
  }

  _scheduleMinHeightRelease(card) {
    clearTimeout(this._minHeightReleaseTimer);
    const generation = ++this._renderGeneration;
    this._minHeightReleaseTimer = setTimeout(() => {
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          if (this._renderGeneration === generation && card && card.isConnected) {
            card.style.minHeight = "";
          }
        });
      });
    }, 150);
  }

  _render() {
    if (!this._config || !this.shadowRoot) return;

    let card = this.shadowRoot.getElementById("aquard-card");
    
    if (!card) {
      this.shadowRoot.innerHTML = `
        <style>${styles}</style>
        <ha-card id="aquard-card"></ha-card>
      `;
      card = this.shadowRoot.getElementById("aquard-card");
    }

    const currentHeight = card.getBoundingClientRect().height;
    if (currentHeight > 50) {
      card.style.minHeight = `${currentHeight}px`;
    }

    if (!hasMeaningfulEntities(this._config)) {
      card.className = "setup-card";
      card.innerHTML = `
        <div class="setup-state" role="status">
          <ha-icon icon="mdi:water-cog-outline" aria-hidden="true"></ha-icon>
          <div><h2>Aquard Einrichtung</h2><p>Waehle deine Spa-Entities in der Kartenkonfiguration aus, um zu starten.</p></div>
        </div>`;

      this._scheduleMinHeightRelease(card);
      return;
    }

    card.className = "";
    const entities = this._config.entities;
    const hasTemperatureSource = Boolean(entities.water_temperature || entities.climate);
    const temperature = readCurrentTemperature(this._hass, entities);
    const power = readSwitch(this._hass, entities.power);
    const isHeating = isHeatingActive(this._hass, entities);
    const autoObj = entities.auto_mode ? this._hass?.states?.[entities.auto_mode] : null;
    const manualObj = entities.manual_mode ? this._hass?.states?.[entities.manual_mode] : null;
    const autoActive = autoObj?.state === "on";
    const manualActive = manualObj?.state === "on";
    const showModeSwitch = Boolean(entities.auto_mode || entities.manual_mode);
    const timeToReadyObj = entities.time_to_ready ? this._hass?.states?.[entities.time_to_ready] : null;
    const timeToReadyFormatted = timeToReadyObj ? renderTimeToReadyText(timeToReadyObj.state) : null;
    const configuredMeasurements = Object.fromEntries(METRICS
      .filter(([key]) => Boolean(entities[key]))
      .map(([key]) => [key, this._hass?.states?.[entities[key]]?.state]));
    const waterQuality = evaluateSpaWaterQuality(configuredMeasurements, buildWaterQualityProfile(this._config));
    const hideUnavailable = this._config?.hide_unavailable === true;
    const controls = [
      [this._label("power"), power, "power", entities.power, false, false],
      [this._label("filter"), readSwitch(this._hass, entities.filter), "filter", entities.filter, false, false],
      [this._label("heater"), readEntity(this._hass, entities.heater), "heater", entities.heater, false, true],
      [this._label("bubbles"), readEntity(this._hass, entities.bubbles), "bubbles", entities.bubbles, true, false],
    ];
    const targetControl = resolveTargetTemperature(this._hass, entities.climate);
    const displayedTarget = this._pendingTarget && this._pendingTarget.entityId === targetControl?.entityId
      ? this._pendingTarget.value
      : targetControl?.target;
    const displayControl = targetControl ? { ...targetControl, target: displayedTarget } : undefined;
    const modes = Object.fromEntries(["water_status", "temperature", "actions", "measurements", "controls", "details"].map((id) => [id, getComponentMode(this._config, id)]));
    const waterVisible = isComponentVisible(this._config, "water_status");
    const hasMeasurements = METRICS.some(([key]) => Boolean(entities[key]));
    const hasWaterStatus = hasMeasurements && waterQuality.score !== null;
    const hasControls = controls.some((control) => Boolean(control[3]));
    const actionsMarkup = renderActions({ mode: modes.actions, hasStatus: hasWaterStatus, standalone: !waterVisible });
    const waterStatusMarkup = hasWaterStatus ? renderWaterStatus({ status: waterQuality.status, mode: modes.water_status, actions: waterVisible ? actionsMarkup : "" }) : "";
    const temperatureMarkup = renderTemperature({ mode: modes.temperature, reading: temperature, configured: hasTemperatureSource, targetControl: displayControl ? this._renderTargetControl(displayControl) : "", targetValue: displayControl?.target ?? null, isHeating, timeToReady: timeToReadyFormatted });
    const heroMarkup = waterStatusMarkup || temperatureMarkup || (!waterVisible ? actionsMarkup : "")
      ? `<div class="hero-grid${waterStatusMarkup && temperatureMarkup ? "" : " hero-grid--focused"}">${(waterStatusMarkup || temperatureMarkup) ? WATER_LINE_DECORATION : ""}${waterStatusMarkup}${temperatureMarkup}${!waterVisible ? actionsMarkup : ""}</div>`
      : "";

    const preheat = readPreheat(this._hass, entities);
    const heatTimer = readHeatTimer(this._hass, entities.heat_timer);
    const filterTimer = readHeatTimer(this._hass, entities.filter_timer);
    const durationReading = entities.heat_duration ? readEntity(this._hass, entities.heat_duration, { numeric: true }) : null;
    const durationValue = durationReading && durationReading.availabilityClass === "available" ? Math.round(Number(durationReading.stateObj.state)) : null;
    const preheatMarkup = renderPreheat({ entities, preheat, title: this._label("preheatTitle") });
    const timedHeatingMarkup = renderTimedHeating({ entities, timer: heatTimer, durationValue, title: this._label("timedHeatingTitle") });
    const dosing = computeDosing(this._hass, entities, this._config);

    const timersArray = [preheatMarkup, timedHeatingMarkup].filter(Boolean);
    const timersMarkup = timersArray.length > 0 ? `<div class="timers-grid">${timersArray.join("")}</div>` : "";

    const anyTimerActive = heatTimer.active || filterTimer.active;
    if (anyTimerActive && !this._countdownTicker) {
      this._countdownTicker = setInterval(() => this._tickCountdowns(), 1000);
    } else if (!anyTimerActive && this._countdownTicker) {
      clearInterval(this._countdownTicker);
      this._countdownTicker = null;
    }

    card.innerHTML = `
        ${renderDetails({ mode: modes.details, name: this._config.name, availabilityClass: temperature.availabilityClass, showAvailability: hasTemperatureSource, autoActive, manualActive, showModeSwitch })}
      <main>
        ${heroMarkup}
        ${renderMeasurements({ mode: modes.measurements, hasMeasurements: hasMeasurements && shouldShowSensorInformation(this._config) })}
        ${renderControls({ mode: modes.controls, hasControls })}
        ${timersMarkup}
      </main>
    `;

    this._setText(".brand-name", this._config.name || UI_TEXT.brand);
    this._setText(".brand-context", UI_TEXT.dashboard);
    this._setText(".header-availability-text", temperature.availabilityClass === "available" ? UI_TEXT.available : temperature.availability);
    this._setText(".status-headline", WATER_STATUS_TEXT[waterQuality.status]);
    this._setText(".status-score", waterQuality.score === null ? "" : `${waterQuality.score}% ${UI_TEXT.quality}`);
    this._setText(".status-action", WATER_ACTION_TEXT[waterQuality.status]);
    this._setText(".status-support-text", WATER_MESSAGE_TEXT[waterQuality.messageKey]);
    this._setText(".temperature-label", this._label("waterTemperature"));
    this._setTemperature(temperature.value);
    this._setText(".temperature-unit", temperature.unit);
    if (displayControl && temperatureMarkup) {
      const decreaseButton = this.shadowRoot.querySelector('[data-target-direction="-1"]');
      const increaseButton = this.shadowRoot.querySelector('[data-target-direction="1"]');
      decreaseButton.addEventListener("click", () => this._adjustTargetTemperature(-1));
      increaseButton.addEventListener("click", () => this._adjustTargetTemperature(1));
    }

    const autoBtn = this.shadowRoot.querySelector('[data-mode-action="auto"]');
    const manualBtn = this.shadowRoot.querySelector('[data-mode-action="manual"]');
    if (autoBtn) autoBtn.addEventListener("click", () => this._toggleMode("auto"));
    if (manualBtn) manualBtn.addEventListener("click", () => this._toggleMode("manual"));

    const droplet = this.shadowRoot.querySelector(".temperature-gauge-droplet");
    if (droplet && entities.climate) {
      droplet.style.cursor = "pointer";
      droplet.setAttribute("role", "button");
      droplet.setAttribute("tabindex", "0");
      droplet.setAttribute("aria-label", UI_TEXT.climateTarget);
      droplet.addEventListener("click", () => this._showMoreInfo(entities.climate));
      droplet.addEventListener("keydown", (event) => { if (event.key === "Enter" || event.key === " ") { event.preventDefault(); this._showMoreInfo(entities.climate); } });
    }

    const equipmentGrid = this.shadowRoot.querySelector(".equipment-grid");
    if (equipmentGrid) for (const control of controls) {
      if (!control[3]) continue;
      if (hideUnavailable && control[1].availabilityClass !== "available") continue;
      const isFilterControl = control[2] === "filter" && (entities.filter_timer || entities.filter_quick_start);
      equipmentGrid.append(isFilterControl ? this._createFilterTile(...control, filterTimer, entities) : this._createEquipmentTile(...control));
    }
    if (equipmentGrid) equipmentGrid.dataset.count = String(equipmentGrid.childElementCount);

    const wpHeatingPanel = this.shadowRoot.querySelector('[data-component="timed_heating"]');
    if (wpHeatingPanel) {
      const startButton = wpHeatingPanel.querySelector('[data-wp-action="start"]');
      const stopButton = wpHeatingPanel.querySelector('[data-wp-action="stop"]');
      const decButton = wpHeatingPanel.querySelector('[data-wp-action="dec"]');
      const incButton = wpHeatingPanel.querySelector('[data-wp-action="inc"]');
      const heatCountdown = wpHeatingPanel.querySelector(".wp-countdown");
      if (startButton) startButton.addEventListener("click", () => this._triggerHeatScript(entities.heat_start));
      if (stopButton) stopButton.addEventListener("click", () => this._triggerHeatScript(entities.heat_stop));
      if (decButton) decButton.addEventListener("click", () => this._adjustHeatDuration(entities.heat_duration, -10));
      if (incButton) incButton.addEventListener("click", () => this._adjustHeatDuration(entities.heat_duration, 10));
      if (heatCountdown && entities.heat_timer) {
        heatCountdown.style.cursor = "pointer";
        heatCountdown.addEventListener("click", () => this._showMoreInfo(entities.heat_timer));
      }
    }

    const metricGrid = this.shadowRoot.querySelector(".metric-grid");
    if (metricGrid) for (const [key, label, icon] of METRICS) {
      if (!entities[key]) continue;
      const metricReading = readEntity(this._hass, entities[key], { numeric: true });
      if (hideUnavailable && metricReading.availabilityClass !== "available") continue;
      metricGrid.append(this._createMetric(
        this._label(key, label),
        this._icon(key),
        metricReading,
        waterQuality.measurements[key],
        buildDosingCornerBadge(key, dosing),
      ));
    }
    if (metricGrid) metricGrid.dataset.count = String(metricGrid.childElementCount);

    this._scheduleMinHeightRelease(card);
  }

  _tickCountdowns() {
    const entities = this._config?.entities;
    if (!entities || !this.shadowRoot) return;
    const heatTimer = readHeatTimer(this._hass, entities.heat_timer);
    const filterTimer = readHeatTimer(this._hass, entities.filter_timer);
    if (!heatTimer.active && !filterTimer.active) {
      clearInterval(this._countdownTicker);
      this._countdownTicker = null;
      this._render();
      return;
    }
    const heatCountdown = this.shadowRoot.querySelector('[data-component="timed_heating"] .wp-countdown');
    if (heatCountdown) heatCountdown.textContent = formatRemaining(heatTimer.remainingSeconds);
    const filterCountdown = this.shadowRoot.querySelector('[data-wp-filter-countdown]');
    if (filterCountdown) filterCountdown.textContent = formatRemaining(filterTimer.remainingSeconds);
  }

  _createFilterTile(label, reading, icon, entityId, allowSelect, allowClimate, timer, entities) {
    const stateObj = entityId ? this._hass?.states?.[entityId] : undefined;
    const action = getControlAction(entityId, stateObj, allowSelect, allowClimate);
    const active = isControlActive(stateObj);
    const pending = this._pendingControls.has(entityId);
    const running = timer.active || timer.paused;

    const wrap = document.createElement("div");
    wrap.className = `equipment-tile equipment-tile--split ${reading.availabilityClass}${active ? " active" : ""}${pending ? " pending" : ""}`;

    const cornerHtml = running
      ? `<span class="equipment-corner-countdown" data-wp-filter-countdown>${formatRemaining(timer.remainingSeconds)}</span>`
      : `<button type="button" class="equipment-corner-action" data-wp-action="filter-start" ${entities.filter_quick_start ? "" : "disabled"} aria-label="${UI_TEXT.quickFilterStart}">${UI_TEXT.quickFilterStart}</button>`;

    wrap.innerHTML = `<button type="button" class="equipment-tile-main" aria-pressed="${active}" aria-busy="${pending}" ${!action || pending ? "disabled" : ""}><div class="equipment-icon equipment-icon-${icon}" aria-hidden="true"><ha-icon icon="${this._icon(icon)}"></ha-icon></div><div class="equipment-copy"><div class="equipment-name"></div><div class="equipment-value"></div></div><span class="status-dot"></span></button>${cornerHtml}`;

    wrap.querySelector(".equipment-name").textContent = label;
    wrap.querySelector(".equipment-value").textContent = reading.availabilityClass === "available" ? translateEquipmentValue(reading.value) : reading.value;
    wrap.querySelector(".equipment-tile-main").setAttribute("aria-label", `${label}: ${wrap.querySelector(".equipment-value").textContent}`);
    wrap.querySelector(".equipment-tile-main").addEventListener("click", () => this._activateControl(entityId, allowSelect, allowClimate));
    const cornerButton = wrap.querySelector('[data-wp-action="filter-start"]');
    if (cornerButton) cornerButton.addEventListener("click", (event) => { event.stopPropagation(); this._triggerHeatScript(entities.filter_quick_start); });
    const cornerCountdown = wrap.querySelector("[data-wp-filter-countdown]");
    if (cornerCountdown && entities.filter_timer) {
      cornerCountdown.style.cursor = "pointer";
      cornerCountdown.addEventListener("click", (event) => { event.stopPropagation(); this._showMoreInfo(entities.filter_timer); });
    }
    return wrap;
  }

    _createEquipmentTile(label, reading, icon, entityId, allowSelect, allowClimate) {
    const stateObj = entityId ? this._hass?.states?.[entityId] : undefined;
    const action = getControlAction(entityId, stateObj, allowSelect, allowClimate);
    const active = isControlActive(stateObj);
    const pending = this._pendingControls.has(entityId);
    const row = document.createElement("button");
    row.type = "button";
    row.className = `equipment-tile ${reading.availabilityClass}${active ? " active" : ""}${pending ? " pending" : ""}`;
    row.disabled = !action || pending;
    row.setAttribute("aria-pressed", String(active));
    row.setAttribute("aria-busy", String(pending));
    row.innerHTML = `<div class="equipment-icon equipment-icon-${icon}" aria-hidden="true"><ha-icon icon="${this._icon(icon)}"></ha-icon></div><div class="equipment-copy"><div class="equipment-name"></div><div class="equipment-value"></div></div><span class="status-dot"></span>`;
    row.querySelector(".equipment-name").textContent = label;
    row.querySelector(".equipment-value").textContent = reading.availabilityClass === "available" ? translateEquipmentValue(reading.value) : reading.value;
    row.setAttribute("aria-label", `${label}: ${row.querySelector(".equipment-value").textContent}`);
    row.addEventListener("click", () => this._activateControl(entityId, allowSelect, allowClimate));
    return row;
  }

  async _activateControl(entityId, allowSelect, allowClimate = false) {
    if (this._pendingControls.has(entityId)) return;
    const action = getControlAction(entityId, this._hass?.states?.[entityId], allowSelect, allowClimate);
    if (!action || typeof this._hass?.callService !== "function") return;

    this._pendingControls.add(entityId);
    this._render();
    try {
      await this._hass.callService(action.domain, action.service, action.data);
    } catch (error) {
      console.error(`Aquard could not control ${entityId}`, error);
    } finally {
      this._pendingControls.delete(entityId);
      this._render();
    }
  }

  _label(key, fallback) {
    return this._config?.labels?.[key] || fallback || UI_TEXT[key];
  }

  _icon(key) {
    return this._config?.icons?.[key] || EQUIPMENT_ICONS[key] || METRIC_ICON_FALLBACKS[key];
  }

  _showMoreInfo(entityId) {
    if (!entityId) return;
    const event = new CustomEvent("hass-more-info", {
      bubbles: true,
      composed: true,
      detail: { entityId },
    });
    this.dispatchEvent(event);
  }

  async _toggleMode(target) {
    const entities = this._config?.entities;
    if (!entities || typeof this._hass?.callService !== "function") return;
    const autoEntity = entities.auto_mode;
    const manualEntity = entities.manual_mode;
    if (target === "auto" && autoEntity) {
      await this._hass.callService("input_boolean", "turn_on", { entity_id: autoEntity });
      if (manualEntity) await this._hass.callService("input_boolean", "turn_off", { entity_id: manualEntity });
    } else if (target === "manual" && manualEntity) {
      await this._hass.callService("input_boolean", "turn_on", { entity_id: manualEntity });
      if (autoEntity) await this._hass.callService("input_boolean", "turn_off", { entity_id: autoEntity });
    }
  }

  async _triggerHeatScript(entityId) {
    if (!entityId || this._pendingHeatAction || typeof this._hass?.callService !== "function") return;
    const [domain, objectId] = String(entityId).split(".", 2);
    if (domain !== "script" || !objectId) return;
    this._pendingHeatAction = true;
    try {
      await this._hass.callService("script", objectId, {});
    } catch (error) {
      console.error(`Aquard (Whirlpool) konnte Skript ${entityId} nicht ausfuehren`, error);
    } finally {
      this._pendingHeatAction = false;
      this._render();
    }
  }

  async _adjustHeatDuration(entityId, deltaMinutes) {
    if (!entityId || typeof this._hass?.callService !== "function") return;
    const stateObj = this._hass?.states?.[entityId];
    if (!stateObj) return;
    const current = Number(stateObj.state);
    if (!Number.isFinite(current)) return;
    const min = Number.isFinite(Number(stateObj.attributes?.min)) ? Number(stateObj.attributes.min) : 5;
    const max = Number.isFinite(Number(stateObj.attributes?.max)) ? Number(stateObj.attributes.max) : 480;
    const next = Math.min(max, Math.max(min, current + deltaMinutes));
    if (next === current) return;
    try {
      await this._hass.callService("input_number", "set_value", { entity_id: entityId, value: next });
    } catch (error) {
      console.error(`Aquard (Whirlpool) konnte Dauer ${entityId} nicht setzen`, error);
    }
  }

  _renderTargetControl(control) {
    const formatted = formatTargetTemperature(control.target, control.unit, control.step);
    const pendingDirection = this._pendingTarget?.direction;
    const decreaseDisabled = !getTargetTemperatureAdjustment(control, -1);
    const increaseDisabled = !getTargetTemperatureAdjustment(control, 1);
    return `<div class="target-control"><div class="target-label">${UI_TEXT.climateTarget}</div><div class="target-control-row">
      <button class="target-button${pendingDirection === -1 ? " pending" : ""}" data-target-direction="-1" aria-label="Zieltemperatur verringern" ${decreaseDisabled ? "disabled" : ""}>${renderTargetArrow("decrease")}</button>
      <div class="target-display"><span class="target-number">${formatted.value}</span><span class="target-unit">${formatted.unit}</span></div>
      <button class="target-button${pendingDirection === 1 ? " pending" : ""}" data-target-direction="1" aria-label="Zieltemperatur erhoehen" ${increaseDisabled ? "disabled" : ""}>${renderTargetArrow("increase")}</button>
    </div></div>`;
  }

  _adjustTargetTemperature(direction) {
    const entityId = this._config?.entities?.climate;
    const control = resolveTargetTemperature(this._hass, entityId);
    if (!control || typeof this._hass?.callService !== "function") return;
    const localControl = this._pendingTarget?.entityId === entityId
      ? { ...control, target: this._pendingTarget.value }
      : control;
    const adjustment = getTargetTemperatureAdjustment(localControl, direction);
    if (!adjustment) return;

    this._clearTargetConfirmationTimer();
    this._pendingTarget = {
      entityId,
      value: adjustment.temperature,
      direction,
      phase: "debounce",
    };
    this._render();

    clearTimeout(this._targetDebounceTimer);
    this._targetDebounceTimer = setTimeout(() => this._flushTargetTemperature(), TARGET_DEBOUNCE_MS);
  }

  async _flushTargetTemperature() {
    clearTimeout(this._targetDebounceTimer);
    this._targetDebounceTimer = null;
    const request = this._pendingTarget;
    if (!request || request.phase !== "debounce" || typeof this._hass?.callService !== "function") return;

    request.phase = "confirming";
    this._render();
    try {
      await this._hass.callService("climate", "set_temperature", {
        entity_id: request.entityId,
        temperature: request.value,
      });
      if (this._pendingTarget !== request) return;
      this._targetConfirmationTimer = setTimeout(() => {
        if (this._pendingTarget === request) {
          this._pendingTarget = null;
          this._targetConfirmationTimer = null;
          this._render();
        }
      }, TARGET_CONFIRMATION_TIMEOUT_MS);
    } catch (error) {
      console.error(`Aquard could not set target temperature for ${request.entityId}`, error);
      if (this._pendingTarget === request) {
        this._pendingTarget = null;
        this._render();
      }
    }
  }

  _reconcilePendingTarget() {
    if (!this._pendingTarget) return;
    if (this._pendingTarget.phase === "debounce") return;
    const control = resolveTargetTemperature(this._hass, this._pendingTarget.entityId);
    if (!control || Math.abs(control.target - this._pendingTarget.value) < 0.000001) {
      clearTimeout(this._targetDebounceTimer);
      this._targetDebounceTimer = null;
      this._clearTargetConfirmationTimer();
      this._pendingTarget = null;
    }
  }

  _clearTargetConfirmationTimer() {
    clearTimeout(this._targetConfirmationTimer);
    this._targetConfirmationTimer = null;
  }

  _createMetric(label, icon, reading, evaluation, cornerBadge = "") {
    const tile = document.createElement("article");
    const qualityClass = evaluation?.severity ?? reading.availabilityClass;
    const rangeDirection = evaluation?.range?.direction ?? "neutral";
    const rangeText = UI_TEXT[`range${titleCase(rangeDirection)}`];
    tile.className = `metric-tile ${reading.availabilityClass} quality-${qualityClass} range-${rangeDirection}`;
    tile.innerHTML = `${cornerBadge}<div class="metric-heading"><span class="metric-label"><ha-icon class="metric-icon" aria-hidden="true"></ha-icon><span class="metric-name"></span></span></div><div class="metric-reading-row"><div class="metric-reading"><span class="metric-value"></span><span class="metric-unit"></span></div><span class="metric-quality-mark" aria-hidden="true"><ha-icon icon="mdi:check"></ha-icon></span></div><div class="metric-meter" role="img"><span class="metric-value-marker"></span></div><div class="metric-footer"><div class="metric-quality"></div><span class="metric-state"><span class="status-dot"></span><span class="metric-state-text"></span></span></div>`;
    tile.querySelector(".metric-icon").setAttribute("icon", icon);
    tile.querySelector(".metric-name").textContent = label;
    tile.querySelector(".metric-value").textContent = reading.value;
    tile.querySelector(".metric-unit").textContent = reading.unit;
    tile.querySelector(".metric-state-text").textContent = reading.availability;
    const qualityText = evaluation?.score === null || evaluation?.score === undefined
      ? reading.availability
      : `${evaluation.score}% ${UI_TEXT.quality}`;
    tile.querySelector(".metric-quality").textContent = evaluation ? `${qualityText} · ${rangeText}` : qualityText;
    tile.querySelector(".metric-meter").setAttribute("aria-label", `${label}: ${rangeText}`);
    if (evaluation?.range) {
      tile.style.setProperty("--range-intensity", evaluation.range.intensity);
      tile.style.setProperty("--range-opacity", 0.42 + (evaluation.range.intensity * 0.55));
      tile.style.setProperty("--current-position", `${evaluation.range.currentPosition}%`);
      const meterColor = computeRangeColor(evaluation.range.direction, evaluation.severity, evaluation.range.intensity);
      if (meterColor) tile.style.setProperty("--meter-color", meterColor);
      else tile.style.removeProperty("--meter-color");
    }
    const qualityIcon = evaluation?.severity === "alert" || evaluation?.severity === "action_needed" ? "mdi:exclamation" : evaluation?.severity === "monitor" ? "mdi:eye-outline" : "mdi:check";
    tile.querySelector(".metric-quality-mark ha-icon").setAttribute("icon", qualityIcon);
    return tile;
  }

  _setTemperature(value) {
    const match = String(value).match(/^(-?\d+)([.,]\d+)$/);
    this._setText(".temperature-whole", match ? match[1] : value);
    this._setText(".temperature-decimal", match ? match[2] : "");
  }

  _setText(selector, value) {
    const element = this.shadowRoot.querySelector(selector);
    if (element) element.textContent = value ?? "";
  }
}

console.info("%c AQUARD-WHIRLPOOL %c build 2026-07-25-v12-final-fix ", "color:#fff;background:#18c8f3;font-weight:700", "color:#18c8f3;background:transparent");

if (!customElements.get("aquard-card-whirlpool")) {
  customElements.define("aquard-card-whirlpool", AquardCard);
}

window.customCards = window.customCards || [];
if (!window.customCards.some((card) => card.type === "aquard-card-whirlpool")) {
  window.customCards.push({
    type: "aquard-card-whirlpool",
    name: "Aquard (Whirlpool, Deutsch)",
    description: "Aquard-Fork auf Deutsch mit Kalender-Vorheizen-Status und Heizen-auf-Zeit-Steuerung.",
    preview: true,
  });
}
