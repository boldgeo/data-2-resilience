# Data2Resilience

A collaborative research project developing a data visualization platform for climate resilience analysis in Dortmund. Led by Prof. Dr. Benjamin Bechtel and Prof. Dr. Christian Albert, this joint initiative between [Ruhr University Bochum](https://www.ruhr-uni-bochum.de/en/) and [Leibniz University Hannover](https://www.uni-hannover.de/) aims to improve urban resilience against extreme heat through innovative biometeorological measurement networks and community engagement.

## Project Purpose & Features

The dashboard enables citizens and stakeholders to:

- Monitor real-time heat stress data across Dortmund
- Visualize climate patterns through interactive maps
- Access detailed measurements from weather stations
- Understand thermal comfort metrics through various indicators
- Make informed decisions for climate adaptation

Funded by the [ICLEI Action Fund 2.0](https://iclei.org/activity/iclei-action-fund-2-0/) and [Google.org](https://www.google.org/), this project combines technological innovation with citizen participation to minimize the impact of extreme heat on urban life.

## Application Features

Data2Resilience is a comprehensive platform designed to visualize climate data from various weather stations. The application allows users to:

- View climate data on an interactive map
- Select specific weather stations for detailed analysis
- Compare measurements across different stations
- Visualize data through various charts (line charts, bar charts, histograms)
- Switch between different climate indicators (temperature, humidity, UTCI, etc.)
- View historical data across different time ranges
- Explore heat stress measurements and thermal comfort indices

### Station Selection

Users can select multiple stations to compare data. Selected station IDs are stored in URL parameters for sharing.

### Time Range Selection

- **Hour**: View data for a specific hour
- **Day**: View daily aggregated data (min/max/avg)
- **Range**: View data over a custom date range

### Measurement Units

Various climate indicators can be selected for visualization:

- Temperature (air, dew point, wet bulb)
- Humidity (relative, absolute)
- Pressure (atmospheric)
- Wind (speed, direction)
- Thermal comfort indices (UTCI, PET)
- Precipitation, solar radiation, etc.

### Data Visualization

- Line charts for time series data
- Bar charts for comparing stations
- Histograms for data distribution
- Special visualizations for heat stress and wind direction

### Interactive Product Tour

We use Shepherd.js to provide an interactive guide through the application's features:

```typescript
import { createTour } from '$lib/utils/tourUtil';

const tour = createTour([
	{
		id: 'welcome',
		title: 'Welcome to Data2Resilience',
		text: 'Let us show you around the main features...'
	},
	{
		id: 'map',
		element: '.map-container',
		title: 'Interactive Map',
		text: 'Explore weather stations and heat stress data...'
	}
	// Additional tour steps...
]);
```

---

## Getting Started

### Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/) version 22.13.1 or later (as specified in `.nvmrc`)
- [PNPM](https://pnpm.io/installation) package manager
- [Git](https://git-scm.com/downloads)
- [Docker](https://www.docker.com/get-started) (optional, for containerized deployment)

### Installation

```bash
# Clone the repository
git clone https://github.com/vogelino/data2resilience.git
cd data2resilience

# Use the correct Node.js version (if you have nvm installed)
nvm use

# Install dependencies
pnpm install

# Copy and configure environment variables
cp .env.example .env
```

After copying the environment file, open `.env` in your editor and configure the following required variables:

- `PUBLIC_API_BASE_URL`: Set to the API endpoint (use `https://api.data2resilience.de` for production or your local API for development)
- `PUBLIC_GEOCODING_URL`: Set to the geocoding service endpoint
- `VITE_BASE_URL`: Set to your local development URL (typically `http://localhost:5173`)
- `PUBLIC_ENABLE_HEATATLAS_TIMESLIDER`: Set to `true` to enable the heat stress time slider on the heat atlas page

### Local Development

```bash
# Start development server with hot reloading
pnpm run dev

# Generate i18n types (required after adding or modifying translations)
pnpm run typesafe-i18n

# Check for TypeScript and Svelte errors
pnpm run check

# Lint the codebase
pnpm run lint

# Format code with Prettier
pnpm run format
```

The development server will be available at http://localhost:5173 by default.

### Building for Production

```bash
# Build the application for production deployment
pnpm run build

# Preview the production build locally
pnpm run preview
```

### Docker Deployment

The application includes Docker configuration for containerized deployment:

```bash
# Build the Docker image
docker build -t data2resilience .

# Run the container (exposing port 3000)
docker run -p 3000:3000 data2resilience

# Run with environment variables
docker run -p 3000:3000 \
  -e PUBLIC_API_BASE_URL=https://api.data2resilience.de \
  -e PUBLIC_GEOCODING_URL=https://geoweb1.digistadtdo.de/doris_gdi/geocoder \
  data2resilience
```

### GitHub Actions Release Pipeline

The app is automatically containerized and published to GitHub Container Registry on releases.

To create a new release:

1. Tag your commit with a version:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```
2. Create a GitHub release through the GitHub interface:

   - Go to the repository on GitHub
   - Click on "Releases"
   - Click "Create a new release"
   - Select the tag you created
   - Add release notes
   - Publish the release

3. The GitHub Actions workflow will automatically:
   - Build the application
   - Create a Docker image
   - Push the image to GitHub Container Registry
   - Tag the image with the release version

---

## Technical Architecture

This section describes the technical foundation of the Data2Resilience application.

![simplified-architecture-diagram](https://github.com/user-attachments/assets/dce04604-c999-418f-ae29-b92d93dbaf4e)

_The diagram above shows how the different components of the application interact, with data flowing from the external APIs through our application layers to the user interface._

### Tech Stack

- **Framework**: [SvelteKit](https://kit.svelte.dev/) 2.0 with TypeScript - Serves as the foundation of the application, providing server-side rendering, routing, and integration with other tools. TypeScript adds type safety throughout the codebase.

- **UI Components**: [shadcn-svelte](https://www.shadcn-svelte.com/) - Provides accessible, customizable UI components that are integrated throughout the application interface, ensuring a consistent design system while maintaining flexibility.

- **Data Visualization**:

  - [LayerChart](https://layerchart.com/) - Used for declarative chart composition, allowing complex visualizations to be built through a composable API. This powers the time series charts and comparison visualizations.
  - [D3.js](https://d3js.org/) - Handles lower-level data manipulation and specialized visualizations like wind direction charts and histograms that require custom rendering.

- **Mapping**: [MapLibre GL](https://maplibre.org/) - Powers the interactive map at the core of the application, displaying station locations, heat maps, and allowing users to select stations directly from the map interface.

- **Geocoding**: [DigiStadtDO](https://digistadtdo.de/) Geocoding API - Enables location search functionality within Dortmund, allowing users to find specific addresses or points of interest and see nearby weather stations. The application uses the BKG API at `https://geoweb1.digistadtdo.de/doris_gdi/geocoder` with the Address search endpoint: `/geosearch?query=address&outputformat=JSON`.
  The geocoding service is specifically limited to addresses in and around Dortmund, as the BKG search has been geographically restricted to this area.

- **Data Fetching**: [TanStack Query](https://tanstack.com/query) - Manages server state with intelligent caching, background updates, and optimistic UI updates when fetching climate data from weather station APIs.

- **State Management**:

  - URL State: [sveltekit-search-params](https://github.com/paoloricciuti/sveltekit-search-params) - Synchronizes application state with URL parameters, enabling shareable links that preserve selected stations, time ranges, and visualization settings.
  - UI State: [Svelte stores](https://svelte.dev/docs/svelte-store) - Manages local UI state like sidebar visibility, selected tabs, and temporary user selections with debounced updates to prevent excessive re-renders.

- **Type Safety**: [Zod](https://zod.dev/) - Validates external data at runtime and provides TypeScript type inference, ensuring that API responses match expected schemas before being used in the application.

- **Styling**: [Tailwind CSS](https://tailwindcss.com/) - Handles all styling through utility classes, enabling rapid UI development while maintaining consistent design tokens across the application.

- **Internationalization**: [typesafe-i18n](https://github.com/ivanhofer/typesafe-i18n) - Provides type-safe translations for the entire application, with automatic code generation to ensure translation keys are always valid.

- **Containerization**: [Docker](https://www.docker.com/) - Packages the application and its dependencies into containers for consistent deployment across different environments, with automated builds via GitHub Actions.

- **User Onboarding**: [Shepherd.js](https://shepherdjs.dev/) - Creates interactive guided tours to help new users understand the dashboard's features and functionality, with customized steps for different parts of the application.

### Project Structure

```
src/
├── app.css               # Global CSS with Tailwind imports
├── app.d.ts              # TypeScript type declarations
├── app.html              # Base HTML template
├── error.html            # Error page template
├── hooks.server.ts       # Server-side hooks for SvelteKit
├── i18n/                 # Internationalization
├── lib/                  # Library code
│   ├── components/       # Reusable UI components
│   ├── schemas/          # Data validation schemas
│   ├── stores/           # Svelte stores for state management
│   └── utils/            # Utility functions
├── params/               # URL parameter matchers
└── routes/               # SvelteKit routes
```

### External APIs

The Data2Resilience platform integrates with several external APIs to provide climate data, geocoding, and mapping capabilities.

#### Climate Data API

**Base URL**: `https://api.data2resilience.de`

**Key Endpoints**:

- `/v1/stations/metadata` - Retrieves metadata about all weather stations
- `/v1/data/{stationId}` - Fetches historical data for a specific station
- `/v1/network-snapshot` - Gets measurements across all stations at a point in time
- `/v1/download/{stationId}` - Downloads full station dataset as CSV
- `/tms/singleband/{unit}/{year}/{doy}/{hour}/{z}/{x}/{y}.png` - Tile service for heat stress visualization
- `/tms/metadata/{unit}/{year}/{doy}/{hour}` - Metadata for heat stress visualization
- `/tms/colormap` - Color mapping for heat stress visualization

**Used For**: Fetching weather station data, heat stress information, and climate measurements across Dortmund.

**Implementation**: Centralized in `lib/utils/api.ts` using TanStack Query for data fetching and caching. These endpoints are consumed by various components like `StationsLineChart`, `DailySationsValuesBarChart`, and `MapMeasurementsRasterLayer`.

**Full API documentation**: https://api.data2resilience.de/docs

#### DigiStadtDO Geocoding API

**Base URL**: `https://geoweb1.digistadtdo.de/doris_gdi/geocoder`

**Key Endpoints**:

- `/geosearch?query={address}&outputformat=JSON` - Address search

**Used For**: Location search functionality within the map interface. The geocoding service is specifically limited to addresses in and around Dortmund.

**Implementation**: Integrated in `MapSearchInput.svelte` component with debounced queries and response validation.

```typescript
const GEOCODING_API = 'https://geodaten.digistadtdo.de/api';

async function searchLocation(query: string) {
	const response = await fetch(`${GEOCODING_API}/geocode?q=${encodeURIComponent(query)}`);
	return response.json();
}
```

#### Mapping Services

##### Vector Tiles

**Approach**:
The application uses custom styled vector tiles for the base map, with light and dark mode variants. The map styles are stored in `$lib/stores/mapStyle.ts` as `positronMapStyleDay` and `positronMapStyleNight`.

**Used For**: Base map vector tiles rendered through MapLibre GL.

**Implementation**: Used in `Map.svelte` to provide the base vector tiles for the map interface, with theme-based switching between light and dark styles.

##### Dortmund Orthophotos

**Endpoint**: `https://www.wms.nrw.de/geobasis/wms_nw_dop`

**Used For**: High-resolution orthophoto layer for Dortmund when satellite view is enabled.

**Implementation**: Implemented in `SatelliteRasterLayer.svelte` as an optional satellite/orthophoto view that can be toggled by users.

---

## Implementation Details

### Key Components

- **Header**: Main navigation component (`Header.svelte`)
- **LeftSidebar**: Contains station selection and filters (`LeftSidebar.svelte`)
- **Map**: Interactive map for visualizing location data
- **Charts**: Various data visualization components:
  - `TimeseriesLineChart.svelte`: Line chart for time series data
  - `DailySationsValuesBarChart.svelte`: Bar chart for daily station values
  - `Histogram.svelte`: Histogram for data distribution
  - `WindDirectionChart.svelte`: Specialized chart for wind direction

### Components and UI

We use [shadcn-svelte](https://www.shadcn-svelte.com/) for core UI components:

```bash
# Install new shadcn component
pnpm dlx shadcn-svelte@latest add button
```

### Key Stores

- `mapData.ts`: Manages the stations GeoJSON data
- `stationsStore.ts`: Handles selected stations state
- `uiStore.ts`: UI-related state (sidebar visibility, date ranges, selected units, etc.)

### Data Flow

1. Station data is loaded from the API during the initial page load
2. Users can select stations from the map or the station selector
3. Selected stations are stored in the state and URL parameters
4. When stations are selected, data visualizations are updated
5. Users can change time ranges, measurement units, and other parameters to update visualizations

### State Management

We use a combination of URL-synchronized state and UI state management:

#### URL State with [sveltekit-search-params](https://github.com/paoloricciuti/sveltekit-search-params)

```typescript
import { queryParam, ssp } from 'sveltekit-search-params';

// Define URL parameters with type safety and defaults
const hourQueryParam = queryParam('hour', ssp.number(12));
const unitQueryParam = queryParam('unit', ssp.string('utci'));

// Create derived stores with validation
export const hour = derived(hourQueryParam, (value) =>
	validateQueryParam(value, z.coerce.number(), 12)
);

// Update with debouncing for performance
export const updateHour = debounce((h: number) => {
	hourQueryParam.set(h);
}, 500);
```

#### Derived State Management with [Svelte Stores](https://svelte.dev/docs/svelte-store)

```typescript
// Complex derived states combining multiple parameters
export const formattedTimeConfiguration = derived(
	[datavisType, rangeStartDate, rangeEndDate],
	([type, start, end]) => {
		// Compute formatted values based on multiple parameters
		return computedValue;
	}
);
```

### Data Fetching

[TanStack Query](https://tanstack.com/query) integration for efficient data fetching:

```typescript
import { createQuery } from '@tanstack/svelte-query';

const query = createQuery({
	queryKey: ['data'],
	queryFn: fetchData
});
```

We also use reactive query arguments with our custom utilities:

```typescript
import { reactiveQueryArgs } from '$lib/utils/queryUtils.svelte';

const query = createQuery(
	reactiveQueryArgs(() => ({
		queryKey: ['stations-snapshot', dateKey, unitWithMinMaxAvg, scale],
		queryFn: async () => {
			/* fetch data */
		},
		enabled: Boolean(dateVal && unitWithMinMaxAvg)
	}))
);
```

### Data Visualization

We use [LayerChart](https://layerchart.com/) for building complex, interactive charts through composition:

```typescript
import { Chart, Line, Points } from '@layerchart/svelte';

<Chart data={timeSeriesData}>
  <Line x="time" y="temperature" />
  <Points x="time" y="temperature" />
</Chart>
```

### Internationalization

We use [typesafe-i18n](https://github.com/ivanhofer/typesafe-i18n) for type-safe translations:

- Translations are stored in `src/i18n/{locale}/index.ts`
- Auto-generated types on the fly with `pnpm typesafe-i18n`

Example translation usage:

```typescript
import LL from '$i18n/i18n-svelte';

// In your component
$LL.pages.about.title(); // Type-safe translation
```

### Theme Support

The app supports light, dark, and system theme modes using [tailwindcss](https://tailwindcss.com/docs/dark-mode):

```typescript
// Toggle theme modes
const theme = $state('light' | 'dark' | 'system');
```

Theme preferences are:

- Persisted in localStorage
- Synced with system preferences
- Automatically applied using Tailwind's dark mode classes

---

## Development Workflow

### Adding a New Component

1. Create a new component in `src/lib/components/`
2. Import and use it in relevant pages or parent components
3. Add any necessary state to stores if needed

### Adding a New Measurement Unit

1. Add the unit to the schema in `src/lib/utils/schemas.ts`
2. Add UI labels in the i18n files
3. Configure the color scale in `src/lib/utils/colorScaleUtil.ts`

### Adding a New Page

1. Create a new route in `src/routes/`
2. Create corresponding components for the page
3. Update navigation components if needed

---

## Maintenance Guide

This section outlines the recommended practices for maintaining the Data2Resilience Svelte application.

### Notes on Maintenance and Troubleshooting in the Frontend

#### General Maintenance

- Regular code reviews should be conducted to ensure code quality and consistency
- The application should be tested thoroughly after any significant changes
- Keep documentation updated whenever changes are made to the application structure or functionality
- Run `pnpm run typesafe-i18n` after adding or modifying internationalization strings

#### Troubleshooting Common Issues

- **API Connection Issues**: Check the environment variables in `.env.production` and `.env.preview`. The application depends on `PUBLIC_API_BASE_URL` and `PUBLIC_GEOCODING_URL`.
- **Visualization Problems**: Inspect data processing logic in components. Most visualization issues are related to data transformation before rendering with LayerChart.
- **Map Rendering Issues**: Verify MapLibre GL dependencies and API keys. Map services are primarily managed through MapLibre components.
- **UI Theme Inconsistencies**: Verify that dark/light mode theme variables in `app.css` are being properly applied through the `mode-watcher` library.

### Environment Variables Management

The application relies on several environment variables that need proper configuration in different environments:

#### Core Environment Variables

- `PUBLIC_API_BASE_URL`: Base URL for the Data2Resilience API (e.g., https://api.data2resilience.de)
- `PUBLIC_GEOCODING_URL`: URL for the DigiStadtDO geocoding service
- `VITE_BASE_URL`: Used for base URL configuration in preview and production environments

#### Environment Files

- `.env.production`: Contains production environment variables
- `.env.preview`: Contains preview environment variables for Vercel preview deployments
- `.env.example`: Template for creating new environment files (never contains actual secret values)

#### Best Practices

- Never commit sensitive keys or tokens to the repository
- Use environment-specific variables in `.env.production` and `.env.preview` files
- For local development, create a local `.env` file (which is ignored by git)
- When adding new environment variables, update the `.env.example` file
- Verify environment variable availability in CI/CD pipelines

### How Dependencies Are Updated

Dependencies should be updated regularly to ensure security and access to new features:

1. **Regular Audit**: Run `pnpm audit` monthly to check for security vulnerabilities
2. **Node Version Management**: Update the Node.js version in `.nvmrc` when necessary (currently uses 22.13.1)
3. **Update Process**:

   ```bash
   # Check outdated packages
   pnpm outdated

   # Update packages (minor and patch versions)
   pnpm update

   # For major version updates (use with caution)
   pnpm update -i
   ```

4. **Testing After Updates**: Always verify application functionality after updating dependencies
5. **Package Lock**: Commit the updated `pnpm-lock.yaml` file to ensure consistent installations
6. **UI Components**: For shadcn-svelte components, use the appropriate CLI:
   ```bash
   pnpm dlx shadcn-svelte@latest add [component-name]
   ```

### How Errors Can Be Found and Reported

#### Error Monitoring

- Frontend errors are monitored using validation through Zod schemas in `lib/utils/schemas.ts`
- API responses are validated with schema parsing in `lib/utils/api.ts`

#### Reporting Issues

1. All issues should be reported via GitHub Issues
2. When submitting an issue, include:
   - Detailed description of the problem
   - Steps to reproduce
   - Expected vs actual behavior
   - Browser/environment information
   - Screenshots if applicable
   - Console errors (if any)
3. Tag issues appropriately with relevant components (API, Map, Visualization, etc.)

### Update Cycles and Best Practices

#### Recommended Update Cycle

- **Security Updates**: Apply immediately after testing
- **Dependency Updates**: Monthly review and updates
- **Feature Releases**: Deploy new features once they've passed testing in preview environments

#### Best Practices for Updates

1. **Version Control**:

   - Use feature branches for all changes
   - Create descriptive commit messages
   - Require pull request reviews before merging to main branches

2. **Testing Requirements**:
   - Ensure data visualization components render correctly with test data
   - Verify map integrations work as expected
   - Test internationalization for German language support (base locale)

### Links to API Documentation

- **Data2Resilience API**: [https://api.data2resilience.de/docs](https://api.data2resilience.de/docs)
- **DigiStadtDO Geocoding API**: [https://geoweb1.digistadtdo.de/doris_gdi/geocoder](https://geoweb1.digistadtdo.de/doris_gdi/geocoder)
- **MapLibre GL**: [https://maplibre.org/maplibre-gl-js-docs/api/](https://maplibre.org/maplibre-gl-js-docs/api/)
- **LayerChart**: [https://layerchart.com](https://layerchart.com)
- **TanStack Query**: [https://tanstack.com/query/latest](https://tanstack.com/query/latest)

### State Management Guidelines

When modifying or adding new features, follow these patterns:

- **URL State**: Use `sveltekit-search-params` for any state that should be shareable or bookmarkable
- **UI State**: Use Svelte stores from `$lib/stores` for local UI state
- **API Data**: Use TanStack Query for server state in components

### Internationalization Maintenance

- Base locale is German (`de`) as configured in `.typesafe-i18n.json`
- Run `pnpm run typesafe-i18n` after adding or modifying strings to regenerate type definitions
- Add new translations in `src/i18n/{locale}/index.ts` files

---

## Data Security and Protection

The Data2Resilience application takes data security and protection seriously. Here are some of the measures in place to ensure sensitive data is properly secured:

### Limited Use of Sensitive Data

The application does not collect, store, or process any sensitive personal data. The only user-specific data used are:

- Locale preferences (for internationalization)
- Theme preferences (light/dark mode)
- Last sidebar state

These preferences are stored client-side using web standard localStorage and are not transmitted to any servers.

### No User Authentication

The app does not implement any user authentication or authorization system. There are no user accounts, logins, or server-side user-specific data stored.

### Secure API Communication

All communication with backend APIs is done over HTTPS to encrypt data in transit. The app interacts with the following API endpoints:

- `https://api.data2resilience.de` - for retrieving sensor metadata and measurements
- `https://geoweb1.digistadtdo.de/doris_gdi/geocoder` - for geocoding functionality within Dortmund

No sensitive data is sent to these APIs. Received data is validated using strict Zod schemas before using in the application.

### Limited Client-Side Data

Sensor metadata and measurements are only fetched on-demand as the user interacts with the application. Data is not preloaded or stored client-side beyond the user's current browsing session.

### Environment Variables

Sensitive configuration like API keys are stored in environment variables (`.env` files) which are not checked into source control. Different environment files are used to configure the app for development (`.env`), preview deployments (`.env.preview`) and production (`.env.production`).

### Secure Deployment

The application is currently deployed on Vercel which provides a secure and compliant hosting environment:

- HTTPS enabled on all deployments
- Vercel DDoS protection
- Segregated environments for production and preview deployments
