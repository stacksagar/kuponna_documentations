# Coordinate to Address Conversion

This document explains how the SearchInput component automatically converts coordinates to human-readable addresses.

## Overview

The SearchInput component now includes automatic coordinate-to-address conversion using Google Maps Geocoding API. When coordinates are provided (either as initial values or selected from the map), they are automatically converted to readable addresses.

## How It Works

### 1. Coordinate Detection

The system automatically detects coordinate strings using a flexible regex pattern:

- `25.638108, 88.639201` ✅
- `6.5244, 3.3792` ✅
- `40.7128, -74.0060` ✅
- `51.5074, -0.1278` ✅

### 2. Automatic Conversion

When coordinates are detected:

1. The system calls Google Maps Geocoding API
2. Converts coordinates to formatted address
3. Updates the display to show the human-readable address
4. Falls back to formatted coordinates if conversion fails

### 3. Fallback Handling

If the conversion fails for any reason:

- API key issues
- Network errors
- Invalid coordinates
- API quota exceeded

The system will format the coordinates for better readability (e.g., `25.6381, 88.6392` instead of `25.638108, 88.639201`).

## Usage

### Basic Usage

```tsx
import SearchInput from "@/components/common/forms/SearchInput";

<SearchInput
  showLocationSearch={true}
  initialLocation="25.638108, 88.639201"
  onLocationSelect={(location) => console.log(location)}
/>;
```

### With Coordinates

```tsx
// The component will automatically convert these coordinates to:
// "Dinajpur District, Bangladesh"
<SearchInput initialLocation="25.638108, 88.639201" />
```

### With Address

```tsx
// Regular addresses work as before
<SearchInput initialLocation="Lagos, Nigeria" />
```

## API Requirements

### Google Maps API Key

The component requires a valid Google Maps API key with Geocoding API enabled:

```typescript
// src/config/googleKeys.ts
export const GOOGLE_API_KEY = "your-api-key-here";
```

### Required APIs

- Geocoding API (for coordinate conversion)
- Places API (for autocomplete functionality)
- Maps JavaScript API (for map display)

## Error Handling

### API Errors

- Invalid API key
- Quota exceeded
- Network failures
- Rate limiting

### Fallback Behavior

- Shows formatted coordinates
- Logs errors to console
- Maintains functionality
- User-friendly error messages

## Performance Considerations

### Caching

- Consider implementing caching for frequently used coordinates
- Google Maps API has usage limits
- Monitor API quota usage

### Debouncing

- Coordinate conversion is triggered on component mount
- No unnecessary API calls during typing
- Efficient state management

## Testing

### Demo Component

Use the `CoordinateConversionDemo` component to test the functionality:

```tsx
import CoordinateConversionDemo from "@/components/examples/CoordinateConversionDemo";

// In your page or component
<CoordinateConversionDemo />;
```

### Test Coordinates

- `25.638108, 88.639201` → Dinajpur District, Bangladesh
- `6.5244, 3.3792` → Lagos, Nigeria
- `40.7128, -74.0060` → New York, USA
- `51.5074, -0.1278` → London, UK

## Troubleshooting

### Common Issues

1. **Coordinates not converting**

   - Check Google Maps API key
   - Verify Geocoding API is enabled
   - Check browser console for errors

2. **API quota exceeded**

   - Monitor usage in Google Cloud Console
   - Implement caching if needed
   - Consider upgrading API plan

3. **Network errors**
   - Check internet connection
   - Verify API endpoint accessibility
   - Check CORS settings

### Debug Mode

Enable console logging to see conversion process:

```typescript
// Check browser console for:
// - Coordinate detection
// - API calls
// - Conversion results
// - Error messages
```

## Future Enhancements

### Planned Features

- [ ] Coordinate caching
- [ ] Batch coordinate conversion
- [ ] Offline coordinate support
- [ ] Custom coordinate formats
- [ ] Reverse geocoding for addresses

### Customization Options

- [ ] Custom coordinate patterns
- [ ] Alternative geocoding services
- [ ] Custom fallback formatting
- [ ] Localization support

## Support

For issues or questions:

1. Check browser console for error messages
2. Verify API key configuration
3. Test with demo component
4. Review this documentation
5. Check Google Maps API status
