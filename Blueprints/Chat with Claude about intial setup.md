As I said, I just started using GitHub and I am all about the open source shortcuts now I just found this on GitHub. This is the introduction to its docs and I will provide you with the state management doc in a second.

react-map-gl was first created by Uber's Visualization team, where Mapbox was used as a component to build powerful web tools such as geospatial analytics and self-driving data visualization. To manage the complexity of these applications, we fully embrace React and reactive programming.

The stock mapbox-gl APIs are imperative. That is, you instruct the map to do something (for example, map.flyTo), and it will execute the command at its own pace.

This does not scale when we have many components that need to synchronize with each other. We sometimes render two maps side by side, and when the user interacts with one, update both cameras. We draw React UI outside of the map container, that moves with the camera. We also render WebGL graphic overlays on top of the map, most notably with deck.gl. In these use cases, in order for all components to synchronize correctly, they must have their shared states managed by React. We might store the source of truth in a parent component state, or Redux store, or hooks, and let it propagate down to the map as well as its peers.

Ultimately, in the spirit of the reactive programming paradigm, data always flows down. As long as the map manages its own state, as mapbox-gl is designed to do, we risk the components going out of sync.

react-map-gl creates a fully reactive wrapper for mapbox-gl. The Map component can be fully controlled, that is, the map's camera would never deviate from the props that it's assigned. Read more about this core behavior in state management.

Limitations

In v7.0, react-map-gl was fully rewritten to better align its API with the underlying Mapbox GL JS library. Wherever the reactive usage patterns permits, the wrapper's props and methods are 1:1 mappings to their native API equivelants.

It is possible to call the native methods directly from the Map instance obtained via getMap. However, doing so may result in the map's state to deviate from its props. For example, calling map.setMaxZoom directly will cause the map's constraint settings to differ from the value of the maxZoom prop. Generally speaking, calling a native method is disencouraged if the same thing can be achieved through the React interface. If a third-party plugin does so, then it may lead to some unexpected behaviors.


This is a fantastic find! Using react-map-gl instead of raw mapbox-gl would fit perfectly with our reactive architecture goals. Here's why this aligns well with our base framework plans:

Reactive State Management
Perfect for our panel system that needs to sync with map state
Better for handling marker interactions
Cleaner state synchronization between components
More React-like programming model
Clean Integration
typescript

Copy
interface MapViewState {
  longitude: number;
  latitude: number;
  zoom: number;
  pitch?: number;
  bearing?: number;
}

function MapContainer() {
  const [viewState, setViewState] = useState<MapViewState>({
    longitude: -100,
    latitude: 40,
    zoom: 3.5
  });

  return (
    <Map
      {...viewState}
      onMove={evt => setViewState(evt.viewState)}
      style={{width: '100%', height: '100%'}}
    >
      {/* Our panel system can live here */}
      {/* Markers can be React components */}
    </Map>
  );
}
Benefits for Our Framework
Better React integration
Cleaner state management
More predictable behavior
Better TypeScript support
Easier to maintain
Would you like to:

See the state management doc you mentioned?
Start implementing a basic setup with react-map-gl?
Modify our blueprint to incorporate react-map-gl's patterns?
This would definitely make our base framework more robust and easier to maintain!

 Copy
Retry


K
There are two ways to use a Map:

Uncontrolled: The application sets the initial view state (Camera options) when the map is mounted, and the component automatically makes changes to the view states afterwards. This mode works very similarly to the mapbox-gl Map class.
Controlled: The application manages the view state, and pass it to the map via props. The map invokes a callback with a new view state during user interaction transition, and the application can decide what to do with it. This mode is the most powerful when an application has other components that need to interact with the map, or implements its own user input and data handling logic.
Uncontrolled Map

You may clone a full app configuration for this example here.

import * as React from 'react';
import Map from 'react-map-gl';

function App() {
  return <Map
    initialViewState={{
      longitude: -100,
      latitude: 40,
      zoom: 3.5
    }}
    mapStyle="mapbox://styles/mapbox/streets-v9"
  />;
}
Controlled Map

You may clone a full app configuration for this example here.

import * as React from 'react';
import Map from 'react-map-gl';

function App() {
  const [viewState, setViewState] = React.useState({
    longitude: -100,
    latitude: 40,
    zoom: 3.5
  });

  return <Map
    {...viewState}
    onMove={evt => setViewState(evt.viewState)}
    mapStyle="mapbox://styles/mapbox/streets-v9"
  />;
}
A real-world application likely uses more complicated state flows:

Using map with a state store (Redux) example
Using map with SSR (Next.js) example
Custom Camera Constraints

Map offers props that set basic constraints for the camera, e.g. maxBounds, minZoom, maxPitch. If you need more complicated logic to constrain the camera, you may use it as a controlled component. The following example restricts the map center inside a GeoJSON geofence:

import * as React from 'react';
import Map from 'react-map-gl';

// npm install @turf/turf
import * as turf from '@turf/turf';

// A circle of 5 mile radius of the Empire State Building
const GEOFENCE = turf.circle([-74.0122106, 40.7467898], 5, {units: 'miles'});

function App() {
  const [viewState, setViewState] = React.useState({
    longitude: -100,
    latitude: 40,
    zoom: 3.5
  });

  const onMove = React.useCallback(({viewState}) => {
    const newCenter = [viewState.longitude, viewState.latitude];
    // Only update the view state if the center is inside the geofence
    if (turf.booleanPointInPolygon(newCenter, GEOFENCE)) {
      setViewState(viewState);
    }
  }, [])

  return <Map
    {...viewState}
    onMove={onMove}
    mapStyle="mapbox://styles/mapbox/streets-v9"
  />;
}


This is excellent! For our base framework, we should definitely use the controlled map pattern since we'll need to:

Sync map state with panels
Handle custom interactions
Manage complex state across components
Implement custom constraints
Let me propose a basic structure for our map container:

typescript

Copy
interface ViewState {
  longitude: number;
  latitude: number;
  zoom: number;
  pitch?: number;
  bearing?: number;
}

interface MapContainerProps {
  initialView?: ViewState;
  constraints?: {
    maxBounds?: [number, number, number, number];
    minZoom?: number;
    maxZoom?: number;
    maxPitch?: number;
    geofence?: any; // GeoJSON
  };
}

const DEFAULT_VIEW: ViewState = {
  longitude: -100,
  latitude: 40,
  zoom: 3.5,
  pitch: 0,
  bearing: 0
};

export function MapContainer({ 
  initialView = DEFAULT_VIEW,
  constraints 
}: MapContainerProps) {
  const [viewState, setViewState] = useState<ViewState>(initialView);
  const [isDragging, setIsDragging] = useState(false);

  const handleMove = useCallback(({viewState: newViewState}) => {
    // Apply any constraints
    if (constraints?.geofence) {
      const newCenter = [newViewState.longitude, newViewState.latitude];
      if (!turf.booleanPointInPolygon(newCenter, constraints.geofence)) {
        return;
      }
    }

    setViewState(newViewState);
  }, [constraints]);

  const handleDragStart = useCallback(() => {
    setIsDragging(true);
  }, []);

  const handleDragEnd = useCallback(() => {
    setIsDragging(false);
  }, []);

  return (
    <div className="relative w-full h-full">
      <Map
        {...viewState}
        onMove={handleMove}
        onDragStart={handleDragStart}
        onDragEnd={handleDragEnd}
        mapStyle="mapbox://styles/mapbox/streets-v9"
        {...constraints}
      >
        {/* This is where we'll mount our panel system */}
        <PanelSystem 
          mapState={viewState}
          isDragging={isDragging}
        />

        {/* This is where we'll mount our markers */}
        <MarkerSystem />
      </Map>
    </div>
  );
}
This gives us:

Controlled map state
Support for constraints
Drag state tracking
Clean mounting points for panels and markers
Would you like me to:

Start implementing this base map container?
Show how the panel system would integrate?
Demonstrate how markers would work with this setup?
The beauty of this approach is that everything stays in sync through React's state management.

 Copy
Retry


K
Perfect what other files do we need for our blank slate mapbox to get it up and running?



2 / 2

For a clean initial setup with react-map-gl, we'll need several core files. Here's what we need:

Project Configuration Files
typescript

Copy
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    open: true
  }
});

// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}

// package.json
{
  "name": "map-framework",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-map-gl": "^7.1.0",
    "mapbox-gl": "^2.15.0",
    "@turf/turf": "^6.5.0",
    "tailwindcss": "^3.3.3",
    "@types/node": "^20.4.5",
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7"
  }
}
Core Application Files
typescript

Copy
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// src/App.tsx
import { MapContainer } from './components/map/MapContainer';
import './App.css';

function App() {
  const MAPBOX_TOKEN = import.meta.env.VITE_MAPBOX_TOKEN;

  return (
    <div className="w-screen h-screen">
      <MapContainer mapboxToken={MAPBOX_TOKEN} />
    </div>
  );
}

export default App;
Environment Setup
bash

Copy
# .env
VITE_MAPBOX_TOKEN=your_mapbox_token_here
Styles
css

Copy
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
}

body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.mapboxgl-canvas {
  outline: none;
}

/* src/App.css */
#root {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
Types
typescript

Copy
// src/types/map.ts
export interface ViewState {
  longitude: number;
  latitude: number;
  zoom: number;
  pitch?: number;
  bearing?: number;
}

export interface MapConstraints {
  maxBounds?: [number, number, number, number];
  minZoom?: number;
  maxZoom?: number;
  maxPitch?: number;
  geofence?: any; // GeoJSON
}

// src/types/index.ts
export * from './map';
Development Configuration
javascript

Copy
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}

// postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
Would you like me to:

Show the full MapContainer implementation next?
Add any additional configuration files?
Show how to set up the development environment?
This gives us a clean, minimal setup with:

TypeScript support
Tailwind for styling
React-map-gl ready to use
Environment variables
Proper type definitions