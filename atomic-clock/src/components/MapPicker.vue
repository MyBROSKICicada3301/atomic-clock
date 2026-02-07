<template>
  <div class="map-container">
    <div class="search-container">
      <div class="search-header">
        <h3>Search Location</h3>
        <button v-if="selectedPlace" @click="clearSelection" class="clear-btn">Clear</button>
      </div>
      <div class="search-input-wrapper">
        <gmpx-place-picker 
          placeholder="Search for a city, address, or landmark"
          class="place-picker-input">
        </gmpx-place-picker>
      </div>
      <div v-if="selectedPlace" class="fun-fact-container">
        <div class="fun-fact-header">
          <h4>Fun Fact</h4>
        </div>
        <div v-if="loadingFunFact" class="fun-fact-loading">
          <div class="spinner"></div>
          <span>Discovering something interesting...</span>
        </div>
        <p v-else-if="funFact" class="fun-fact-text">{{ funFact }}</p>
        <p v-else class="fun-fact-error">Unable to load fun fact</p>
      </div>
    </div>
    <gmp-map 
      ref="mapElement"
      :center="{ lat: 12.9629, lng: 77.5775 }"
      :zoom="13" 
      map-id="DEMO_MAP_ID"
      style="height: 500px; width: 100%;">
      <gmp-advanced-marker ref="markerElement"></gmp-advanced-marker>
    </gmp-map>
  </div>
</template>

<script>
export default {
  name: 'MapPicker',
  emits: ['placeSelected'],
  data() {
    return {
      selectedPlace: null,
      funFact: '',
      loadingFunFact: false,
      availableModel: 'gemini-1.5-flash' // Default fallback
    };
  },
  mounted() {
    this.init();
    this.checkAvailableModels();
  },
  methods: {
    async init() {
      await customElements.whenDefined('gmp-map');

      const map = this.$refs.mapElement;
      const marker = this.$refs.markerElement;
      const placePicker = document.querySelector('gmpx-place-picker');
      const infowindow = new google.maps.InfoWindow();

      map.innerMap.setOptions({
        mapTypeControl: false
      });

      placePicker.addEventListener('gmpx-placechange', async () => {
        const place = placePicker.value;

        if (!place.location) {
          window.alert(
            "No details available for input: '" + place.name + "'"
          );
          infowindow.close();
          marker.position = null;
          return;
        }

        // Extract coordinates properly
        const lat = typeof place.location.lat === 'function' ? place.location.lat() : place.location.lat;
        const lng = typeof place.location.lng === 'function' ? place.location.lng() : place.location.lng;
        const locationObj = { lat, lng };

        if (place.viewport) {
          map.innerMap.fitBounds(place.viewport);
        } else {
          map.center = locationObj;
          map.zoom = 17;
        }

        marker.position = locationObj;
        infowindow.setContent(
          `<strong>${place.displayName}</strong><br>
           <span>${place.formattedAddress}</span>`
        );
        infowindow.open(map.innerMap, marker);

        // Get timezone information
        const timezone = await this.getTimezone(lat, lng);
        
        this.selectedPlace = {
          name: place.displayName,
          location: locationObj,
          timezone: timezone
        };
        
        this.$emit('placeSelected', this.selectedPlace);
        
        // Get fun fact about the location
        this.getFunFact(place.displayName);
      });
    },
    formatCoordinates() {
      if (!this.selectedPlace || !this.selectedPlace.location) return '';
      const lat = this.selectedPlace.location.lat.toFixed(6);
      const lng = this.selectedPlace.location.lng.toFixed(6);
      return `${lat}, ${lng}`;
    },
    clearSelection() {
      this.selectedPlace = null;
      this.funFact = '';
      this.loadingFunFact = false;
      const marker = this.$refs.markerElement;
      if (marker) {
        marker.position = null;
      }
      const placePicker = document.querySelector('gmpx-place-picker');
      if (placePicker) {
        placePicker.value = null;
      }
      this.$emit('placeSelected', null);
    },
    async getTimezone(lat, lng) {
      const timestamp = Math.floor(Date.now() / 1000);
      const apiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
      
      try {
        const response = await fetch(
          `https://maps.googleapis.com/maps/api/timezone/json?location=${lat},${lng}&timestamp=${timestamp}&key=${apiKey}`
        );
        const data = await response.json();
        
        if (data.status === 'OK') {
          return data.timeZoneId;
        }
      } catch (error) {
        // console.error('Error fetching timezone:', error);
      }
      
      return null;
    },
    async checkAvailableModels() {
      const apiKey = import.meta.env.VITE_GEMINI_API_KEY;
      const url = `https://generativelanguage.googleapis.com/v1beta/models?key=${apiKey}`;
      
      try {
        const response = await fetch(url);
        const data = await response.json();

        if (!response.ok) {
          // console.error('Error fetching models:', data);
          return;
        }

        // console.log("Available Gemini Models:");
        const contentGenerationModels = [];
        
        data.models.forEach(model => {
          const modelName = model.name.replace('models/', '');
          // console.log(`- ${modelName} (${model.displayName})`);
          
          // Check if model supports generateContent, this was done since I noticed some models like Gemini 1.5 Pro don't support it and cause errors
          if (model.supportedGenerationMethods && 
              model.supportedGenerationMethods.includes('generateContent')) {
            contentGenerationModels.push(modelName);
          }
        });

        // Prefer newer models if available
        if (contentGenerationModels.includes('gemini-2.5-flash')) {
          this.availableModel = 'gemini-2.5-flash';
        } else if (contentGenerationModels.includes('gemini-1.5-flash')) {
          this.availableModel = 'gemini-1.5-flash';
        } else if (contentGenerationModels.length > 0) {
          this.availableModel = contentGenerationModels[0];
        }
        
        // console.log(`Using model: ${this.availableModel}`);
      } catch (error) {
        // console.error('Error checking for available models:', error);
      }
    },
    async getFunFact(placeName) {
      this.loadingFunFact = true;
      this.funFact = '';
      
      const apiKey = import.meta.env.VITE_GEMINI_API_KEY;
      const url = `https://generativelanguage.googleapis.com/v1beta/models/${this.availableModel}:generateContent?key=${apiKey}`;
      
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            contents: [{
              parts: [{
                text: `Give me one interesting and surprising fun fact about ${placeName} related to history. Keep it to 2-3 sentences. Make it engaging and not commonly known.`
              }]
            }]
          })
        });
        
        const data = await response.json();
        
        if (!response.ok) {
          // console.error('API Error:', data);
          this.funFact = 'Unable to fetch fun fact.';
          return;
        }
        
        if (data.candidates && data.candidates[0]?.content?.parts?.[0]?.text) {
          this.funFact = data.candidates[0].content.parts[0].text;
        } else {
          this.funFact = 'No fun fact available at the moment.';
        }
      } catch (error) {
        // console.error('Error fetching fun fact:', error);
        this.funFact = 'Unable to fetch fun fact.';
      } finally {
        this.loadingFunFact = false;
      }
    }
  }
};
</script>

<style scoped>
.map-container {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.search-container {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.search-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.search-header h3 {
  margin: 0;
  color: #333;
  font-size: 1.2rem;
  font-weight: 600;
}

.clear-btn {
  background: #ff4444;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.3s;
}

.clear-btn:hover {
  background: #cc0000;
}

.search-input-wrapper {
  margin-bottom: 1rem;
}

.place-picker-input {
  width: 100%;
  font-size: 1rem;
}

gmpx-place-picker {
  width: 100%;
}

.location-info {
  background: white;
  border-radius: 8px;
  padding: 1rem;
  margin-top: 1rem;
  border-left: 4px solid #4CAF50;
}

.info-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 0;
  border-bottom: 1px solid #eee;
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  font-weight: 600;
  color: #555;
  font-size: 0.95rem;
}

.info-value {
  color: #333;
  font-size: 0.95rem;
  text-align: right;
  max-width: 60%;
  word-break: break-word;
}

.fun-fact-container {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  padding: 1.5rem;
  margin-top: 1.5rem;
  color: white;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.fun-fact-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.fun-fact-icon {
  font-size: 1.5rem;
}

.fun-fact-header h4 {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
}

.fun-fact-text {
  margin: 0;
  line-height: 1.6;
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.95);
}

.fun-fact-loading {
  display: flex;
  align-items: center;
  gap: 1rem;
  color: rgba(255, 255, 255, 0.9);
}

.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.fun-fact-error {
  margin: 0;
  color: rgba(255, 255, 255, 0.7);
  font-style: italic;
}
</style>
