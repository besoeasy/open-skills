# City Distance Skill

Purpose: Calculate line-of-sight and road distances between two cities using free, API-keyless public services and local haversine calculations.

What it does:
- Computes line-of-sight distance using the Haversine formula.
- Uses OpenStreetMap routing endpoint (routing.openstreetmap.de) to compute road distance without an API key.
- Optionally lists intermediate cities by sampling points along the route and reverse-geocoding with Nominatim (free) to find nearby settlements.

Files:
- city_distance_calculator.js — example Node.js script demonstrating the calculations.

Usage:
1. Run `node city_distance_calculator.js` with Node.js 18+ (or install node-fetch for older Node versions).
2. The script prints line-of-sight and road distances.

Notes / Rate limits:
- routing.openstreetmap.de and Nominatim are public services and have usage policies and rate limits. Use respectfully (cache results, avoid heavy automated polling).
- For production-grade use, consider hosting your own OSRM/GraphHopper instance or using a commercial API with SLA.

