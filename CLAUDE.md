# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

City Map Poster Generator - Creates minimalist map posters for any city using OpenStreetMap data. Single Python script that fetches street networks, water features, and parks, then renders them as styled poster images.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Generate a poster
python create_map_poster.py -c "City Name" -C "Country" -t theme_name -d distance_meters

# List available themes
python create_map_poster.py --list-themes

# Quick preview (lower quality, faster)
# Modify dpi=300 to dpi=150 in create_poster() for faster renders
```

## Architecture

The project is a single-file application (`create_map_poster.py`) with this data flow:

```
CLI (argparse) → Geocoding (Nominatim) → Data Fetching (OSMnx) → Rendering (matplotlib) → PNG Output
```

**Key dependencies:**
- `osmnx` - Fetches street networks and geographic features from OpenStreetMap
- `geopy` - Geocoding via Nominatim
- `matplotlib` - Rendering and output

**Global state:** `THEME` dict loaded from JSON, `FONTS` dict loaded at startup

## Rendering Layers (z-order)

```
z=11  Text labels (city, country, coords)
z=10  Gradient fades (top & bottom)
z=3   Roads (via ox.plot_graph)
z=2   Parks
z=1   Water
z=0   Background
```

## Key Functions

| Function | Purpose |
|----------|---------|
| `get_coordinates()` | City → lat/lon via Nominatim |
| `create_poster()` | Main rendering pipeline |
| `get_edge_colors_by_type()` | Road color by OSM highway tag |
| `get_edge_widths_by_type()` | Road width by importance |
| `create_gradient_fade()` | Top/bottom fade overlay |
| `load_theme()` | JSON theme → dict |

## Adding New Features

**New map layer (e.g., railways):**
```python
# In create_poster(), after parks fetch:
railways = ox.features_from_point(point, tags={'railway': 'rail'}, dist=dist)
# Plot before roads:
railways.plot(ax=ax, color=THEME['railway'], linewidth=0.5, zorder=2.5)
```

**New theme property:**
1. Add to theme JSON: `"property_name": "#COLOR"`
2. Use in code: `THEME['property_name']`
3. Add fallback in `load_theme()` default dict

## Theme Structure

Themes are JSON files in `themes/` directory. Required properties:
- `bg`, `text`, `gradient_color` - Background, text, and gradient colors
- `water`, `parks` - Feature colors
- `road_motorway`, `road_primary`, `road_secondary`, `road_tertiary`, `road_residential`, `road_default` - Road hierarchy colors

## Distance Guide

| Distance | Best for |
|----------|----------|
| 4000-6000m | Small/dense cities (Venice, Amsterdam center) |
| 8000-12000m | Medium cities (Paris, Barcelona) |
| 15000-20000m | Large metros (Tokyo, Mumbai) |

## Output

Posters saved to `posters/` as `{city}_{theme}_{YYYYMMDD_HHMMSS}.png` at 300 DPI.
