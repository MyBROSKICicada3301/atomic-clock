# Atomic World Clock

A modern, real-time world clock application with Google Maps integration and AI-powered location facts. View local times across different timezones with millisecond precision.


## Features

- **Real-Time Clock**: High-precision atomic clock with millisecond accuracy
- **Local Time Display**: Shows your current local time and date
- **World Timezone Converter**: Search any location and see its current time instantly
- **Interactive Map**: Visual location picker powered by Google Maps
- **AI-Powered Fun Facts**: Get interesting facts about searched locations using Google Gemini AI
- **Sleek UI**: Departure board style interface with monospace fonts
- **Responsive Design**: Works perfectly on desktop and mobile devices

## Live Demo

Visit the live application: **[https://atomic-world-clock.web.app](https://atomic-world-clock.web.app)**

## Tech Stack

- **Frontend Framework**: Vue.js 3
- **Build Tool**: Vite
- **Styling**: Tailwind CSS utility classes + Custom CSS
- **Maps Integration**: Google Maps JavaScript API + Extended Components
- **AI Integration**: Google Gemini API
- **Hosting**: Firebase Hosting
- **Font**: Share Tech Mono (Google Fonts)

## Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/) (v16 or higher)
- [npm](https://www.npmjs.com/) or [yarn](https://yarnpkg.com/)
- [Firebase CLI](https://firebase.google.com/docs/cli) (for deployment)

## API Keys Required

You'll need the following API keys:

1. **Google Maps API Key**

   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Enable these APIs:
     - Maps JavaScript API
     - Places API
     - Timezone API
   - Create credentials and get your API key
2. **Google Gemini API Key**

   - Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
   - Create an API key for Gemini
3. **Firebase Project**

   - Create a project at [Firebase Console](https://console.firebase.google.com/)
   - Enable Firebase Hosting

## Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/yourusername/atomic-world-clock.git
   cd atomic-world-clock/atomic-clock
   ```
2. **Install dependencies**

   ```bash
   npm install
   ```
3. **Set up environment variables**

   Copy the example env file:

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and add your API keys:

   ```env
   VITE_GOOGLE_MAPS_API_KEY=your_google_maps_api_key
   VITE_GEMINI_API_KEY=your_gemini_api_key
   VITE_HOSTING_URL=your_firebase_hosting_url
   VITE_FIREBASE_PROJECT_ID=your_firebase_project_id
   ```
4. **Update index.html**

   The `index.html` file needs to reference your API key. Vite doesn't process HTML files for env variables, so update it manually or use the build process.

## 🚀 Development

Start the development server:

```bash
npm run dev
```

The app will be available at `http://localhost:5173/`

## 🏗️ Build for Production

Build the production-ready app:

```bash
npm run build
```

The optimized files will be in the `dist/` folder.

Preview the production build locally:

```bash
npm run preview
```

## 🔥 Firebase Deployment

1. **Login to Firebase**

   ```bash
   firebase login
   ```
2. **Initialize Firebase (if not already done)**

   ```bash
   firebase init hosting
   ```

   - Select your Firebase project
   - Set public directory to: `atomic-clock/dist`
   - Configure as single-page app: Yes
3. **Build and Deploy**

   ```bash
   npm run build
   cd ..
   firebase deploy
   ```

Your app will be live at your Firebase Hosting URL!

## 📁 Project Structure

```
atomic-clock/
├── public/              # Static assets
├── src/
│   ├── assets/         # Images, icons
│   ├── components/
│   │   ├── MapPicker.vue      # Map & location picker component
│   │   └── HelloWorld.vue     # Example component
│   ├── App.vue         # Main app component
│   ├── main.js         # App entry point
│   └── style.css       # Global styles
├── index.html          # HTML entry point
├── vite.config.js      # Vite configuration
├── package.json        # Dependencies
├── .env               # Environment variables (not in git)
├── .env.example       # Example env file
└── README.md          # This file
```

## 🎨 Features Details

### Atomic Clock Display

- Displays local time with millisecond precision
- Updates every 10ms for smooth animation
- Shows day of week and full date
- Monospace font for authentic departure board look

### Location Time Converter

- Search any city, address, or landmark
- Automatically fetches timezone information
- Displays both date and time for selected location
- Shows timezone differences at a glance

### Interactive Map

- Powered by Google Maps with advanced markers
- Click on locations to set markers
- Info windows with location details
- Smooth zoom and pan animations

### AI Fun Facts

- Automatically generates interesting facts about searched locations
- Uses Google Gemini Flash model for fast responses
- 2-3 sentence engaging facts
- Beautiful gradient card design with loading animation

## ⚙️ Configuration

### Vite Config (`vite.config.js`)

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    vue({
      template: {
        compilerOptions: {
          isCustomElement: (tag) => tag.startsWith('gmp-') || tag.startsWith('gmpx-')
        }
      }
    })
  ],
})
```

This configuration tells Vue to treat Google Maps custom elements as native elements.

### Firebase Config (`firebase.json`)

```json
{
  "hosting": {
    "public": "atomic-clock/dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

## 🔒 Security Notes

- Never commit your `.env` file to version control
- Keep your API keys secure and regenerate if exposed
- Use environment variables for all sensitive data
- Consider adding API key restrictions in Google Cloud Console
- Enable rate limiting on your Firebase project

## 🐛 Troubleshooting

### Google Maps not loading

- Check that your API key has the correct APIs enabled
- Verify API key restrictions in Google Cloud Console
- Check browser console for error messages

### Fun facts not working

- Verify Gemini API key is correct
- Check that the API endpoint is accessible
- Review browser console for API errors

### Build errors

- Clear node_modules and reinstall: `rm -rf node_modules && npm install`
- Check Node.js version: `node --version` (should be v16+)
- Verify all env variables are set correctly

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 👨‍💻 Author

**Your Name**

- GitHub: [@yourusername](https://github.com/yourusername)
- Website: [yourwebsite.com](https://yourwebsite.com)

## 🙏 Acknowledgments

- Google Maps Platform for mapping services
- Google Gemini AI for fun facts
- Vue.js team for the amazing framework
- Firebase for hosting infrastructure
- Share Tech Mono font for the retro look

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📧 Contact

For questions or support, please open an issue on GitHub.

---

Made with ❤️ using Vue.js and Google APIs
