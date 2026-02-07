<template>
  <div class="min-h-screen bg-gray-100">
    <div class="container mx-auto p-4">
      <div class="mb-8 flex items-center justify-center bg-black text-yellow-400">
        <div class="departure-board shadow-2xl rounded-xl p-8">
          <h1 class="text-2xl font-bold text-center mb-6 tracking-widest">
            ATOMIC TIME
          </h1>

          <div class="time-row">
            <span class="label">LOCAL TIME:</span>
            <span class="time">{{ time }}</span>
          </div>

          <div class="time-row">
            <span class="label">DATE:</span>
            <span class="time">{{ date }}</span>
          </div>

          <div v-if="selectedPlace" class="time-row highlight">
            <span class="label location-label">{{ selectedPlace.name.toUpperCase() }}</span>
            <div class="time-group">
              <span class="time">{{ localTime }}</span>
              <span class="date-small">{{ localDate }}</span>
            </div>
          </div>
        </div>
      </div>

      <div class="bg-white rounded-xl shadow-lg p-6">
        <h2 class="text-2xl font-bold text-center mb-4 text-gray-800">
          Place Picker
        </h2>
        <MapPicker @placeSelected="handlePlaceSelected" />
      </div>
    </div>
  </div>
</template>

<script>
import MapPicker from './components/MapPicker.vue';

export default {
  name: "AtomicClock",
  components: {
    MapPicker
  },
  data() {
    return {
      time: "--:--:--.---",
      date: "---- -- --",
      localTime: "--:--:--.---",
      localDate: "---- -- --",
      timer: null,
      selectedPlace: null,
    };
  },
  methods: {
    updateTime() {
      const now = new Date();
      const pad = (n, z = 2) => String(n).padStart(z, "0");

      // Get local time instead of UTC
      const h = pad(now.getHours());
      const m = pad(now.getMinutes());
      const s = pad(now.getSeconds());
      const ms = pad(now.getMilliseconds(), 3);

      this.time = `${h}:${m}:${s}.${ms}`;
      
      // Format date with day of week
      const days = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];
      const months = ['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC'];
      const dayName = days[now.getDay()];
      const monthName = months[now.getMonth()];
      const dayNum = pad(now.getDate());
      const year = now.getFullYear();
      
      this.date = `${dayName}, ${monthName} ${dayNum} ${year}`;

      // Update local time if a place is selected
      if (this.selectedPlace && this.selectedPlace.timezone) {
        this.updateLocalTime(now);
      }
    },
    updateLocalTime(now) {
      const pad = (n, z = 2) => String(n).padStart(z, "0");
      
      const formatter = new Intl.DateTimeFormat('en-US', {
        timeZone: this.selectedPlace.timezone,
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        hour12: false
      });
      
      const parts = formatter.formatToParts(now);
      const h = parts.find(p => p.type === 'hour').value;
      const m = parts.find(p => p.type === 'minute').value;
      const s = parts.find(p => p.type === 'second').value;
      const ms = pad(now.getMilliseconds(), 3);
      
      this.localTime = `${h}:${m}:${s}.${ms}`;
      
      // Format date for selected place
      const dateFormatter = new Intl.DateTimeFormat('en-US', {
        timeZone: this.selectedPlace.timezone,
        weekday: 'short',
        month: 'short',
        day: '2-digit',
        year: 'numeric'
      });
      
      const dateString = dateFormatter.format(now);
      const dateParts = dateString.split(', ');
      const dayName = dateParts[0].toUpperCase();
      const restOfDate = dateParts[1].replace(/\s+/g, ' ');
      
      this.localDate = `${dayName}, ${restOfDate.toUpperCase()}`;
    },
    handlePlaceSelected(place) {
      this.selectedPlace = place;
      if (place && place.timezone) {
        this.updateLocalTime(new Date());
      } else {
        this.localTime = "--:--:--.---";
        this.localDate = "---- -- --";
      }
    },
  },
  mounted() {
    this.updateTime();
    this.timer = setInterval(this.updateTime, 10);
  },
  beforeUnmount() {
    clearInterval(this.timer);
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap");

.container {
  max-width: 1200px;
}

.departure-board {
  background: #0b0b0b;
  border: 6px solid #1f1f1f;
  font-family: "Share Tech Mono", monospace;
  min-width: 360px;
}

.time-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1.8rem;
  padding: 0.75rem 0;
  border-bottom: 2px dotted #333;
  gap: 1rem;
}

.time-row.highlight {
  border-bottom: 2px solid #00ff66;
  margin-top: 1rem;
  padding-top: 1rem;
}

.time-row.highlight .label {
  color: #00ff66;
  font-size: 1.2rem;
  text-transform: uppercase;
  font-weight: 600;
}

.time-row.highlight .time {
  color: #00ff66;
  font-weight: 600;
}

.label {
  color: #888;
  white-space: nowrap;
  min-width: fit-content;
}

.location-label {
  white-space: normal;
  word-break: break-word;
  max-width: 50%;
  line-height: 1.3;
}

.time {
  letter-spacing: 0.15em;
  white-space: nowrap;
  text-align: right;
}

.time-group {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.25rem;
}

.date-small {
  font-size: 0.9rem;
  letter-spacing: 0.1em;
  color: #00ff66;
  opacity: 0.85;
}

.status-row {
  display: flex;
  justify-content: space-between;
  font-size: 1.4rem;
}

.status {
  color: #888;
}

.on-time {
  color: #00ff66;
  font-weight: bold;
  animation: blink 1.5s infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}
</style>
