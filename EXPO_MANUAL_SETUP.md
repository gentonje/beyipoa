
# Expo Mobile App Manual Setup Guide

This document contains all the manual changes needed to complete the Expo mobile app setup that couldn't be automatically implemented.

## 1. Required Assets

Create the following asset files in `expo/assets/`:

### expo/assets/icon.png
- Size: 1024x1024 pixels
- Format: PNG with transparency
- Purpose: App icon for all platforms

### expo/assets/splash.png
- Size: 1284x2778 pixels (iPhone 14 Pro Max resolution)
- Format: PNG
- Purpose: Launch screen image

### expo/assets/favicon.png
- Size: 32x32 pixels
- Format: PNG
- Purpose: Web favicon when running in browser

### expo/assets/adaptive-icon.png
- Size: 1024x1024 pixels
- Format: PNG with transparency
- Purpose: Android adaptive icon

### expo/src/assets/placeholder.png
- Size: 400x300 pixels
- Format: PNG
- Purpose: Placeholder image for products without images
- Content: Simple gray background with "No Image" text

## 2. Environment Variables

Create `expo/.env` file:

```env
# Supabase Configuration
EXPO_PUBLIC_SUPABASE_URL=your_supabase_url_here
EXPO_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key_here

# App Configuration
EXPO_PUBLIC_APP_NAME=YourAppName
EXPO_PUBLIC_APP_VERSION=1.0.0
```

## 3. Additional Configuration Files

### expo/eas.json (for EAS Build)
```json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {}
  },
  "submit": {
    "production": {}
  }
}
```

### expo/.gitignore (additional entries)
```
# Expo
.expo/
dist/
web-build/

# Dependencies
node_modules/

# Environment
.env
.env.local
.env.production

# Build artifacts
*.orig.*
*.jks
*.p8
*.p12
*.key
*.mobileprovision

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage/

# nyc test coverage
.nyc_output

# Dependency directories
node_modules/
jspm_packages/

# TypeScript cache
*.tsbuildinfo

# Optional npm cache directory
.npm

# Optional eslint cache
.eslintcache

# Optional REPL history
.node_repl_history

# Output of 'npm pack'
*.tgz

# Yarn Integrity file
.yarn-integrity

# Stores VSCode versions used for testing VSCode extensions
.vscode-test

# yarn v2
.yarn/cache
.yarn/unplugged
.yarn/build-state.yml
.yarn/install-state.gz
.pnp.*
```

## 4. Installation Commands

Run these commands in the `expo/` directory:

```bash
# Install dependencies
npm install

# Install Expo CLI globally (if not already installed)
npm install -g @expo/cli

# Install iOS dependencies (macOS only)
npx pod-install ios

# For development
npm start

# For iOS simulator
npm run ios

# For Android emulator
npm run android

# For web (development)
npm run web
```

## 5. Platform-Specific Setup

### iOS Setup (macOS required)
1. Install Xcode from App Store
2. Install Xcode Command Line Tools:
   ```bash
   xcode-select --install
   ```
3. Install CocoaPods:
   ```bash
   sudo gem install cocoapods
   ```

### Android Setup
1. Install Android Studio
2. Set up Android SDK and emulator
3. Add Android SDK to PATH:
   ```bash
   export ANDROID_HOME=$HOME/Library/Android/sdk
   export PATH=$PATH:$ANDROID_HOME/emulator
   export PATH=$PATH:$ANDROID_HOME/tools
   export PATH=$PATH:$ANDROID_HOME/tools/bin
   export PATH=$PATH:$ANDROID_HOME/platform-tools
   ```

## 6. Supabase Configuration

Update your Supabase project settings:

### Row Level Security (RLS) Policies
Ensure these policies exist in your Supabase dashboard:

```sql
-- Products table
CREATE POLICY "Public can view published products" ON products
FOR SELECT USING (product_status = 'published');

CREATE POLICY "Users can manage own products" ON products
FOR ALL USING (auth.uid() = user_id);

-- Profiles table
CREATE POLICY "Users can view all profiles" ON profiles
FOR SELECT USING (true);

CREATE POLICY "Users can update own profile" ON profiles
FOR UPDATE USING (auth.uid() = id);

-- Cart items
CREATE POLICY "Users can manage own cart" ON cart_items
FOR ALL USING (auth.uid() = user_id);

-- Orders
CREATE POLICY "Users can view own orders" ON orders
FOR SELECT USING (auth.uid() = user_id);

-- Notifications
CREATE POLICY "Users can view own notifications" ON notifications
FOR SELECT USING (auth.uid() = user_id);
```

### Storage Bucket Policies
Configure storage buckets in Supabase:

```sql
-- Product images bucket
INSERT INTO storage.buckets (id, name, public) 
VALUES ('product-images', 'product-images', true);

-- Profile avatars bucket
INSERT INTO storage.buckets (id, name, public) 
VALUES ('avatars', 'avatars', true);
```

## 7. Push Notifications Setup

### iOS Push Notifications
1. Create Apple Developer account
2. Generate APNs certificate
3. Add to Expo dashboard under project settings

### Android Push Notifications
1. Create Firebase project
2. Download `google-services.json`
3. Add FCM server key to Expo dashboard

### Configure in app.json
```json
{
  "expo": {
    "notification": {
      "icon": "./assets/notification-icon.png",
      "color": "#6366f1",
      "sounds": ["./assets/notification-sound.wav"]
    }
  }
}
```

## 8. Database Schema Verification

Ensure these tables exist in your Supabase database:

- `profiles` - User profiles
- `products` - Product listings
- `product_images` - Product image references
- `categories` - Product categories
- `cart_items` - Shopping cart items
- `orders` - Order records
- `order_items` - Order line items
- `reviews` - Product reviews
- `notifications` - User notifications
- `countries` - Country data
- `districts` - Geographic districts
- `currencies` - Currency definitions
- `account_types` - User account types

## 9. Testing Checklist

After setup, test these features:

### Authentication
- [ ] User registration
- [ ] User login
- [ ] Password reset
- [ ] Session persistence

### Products
- [ ] Product listing
- [ ] Product search and filters
- [ ] Product details view
- [ ] Add/edit products
- [ ] Image upload

### Shopping
- [ ] Add to cart
- [ ] Cart management
- [ ] Checkout process
- [ ] Order history

### Social Features
- [ ] Product reviews
- [ ] Wishlist functionality
- [ ] Chat/messaging

### Admin Features
- [ ] User management
- [ ] Product moderation
- [ ] Analytics dashboard

### Mobile Features
- [ ] Camera integration
- [ ] Push notifications
- [ ] Offline storage
- [ ] Navigation

## 10. Common Issues and Solutions

### Build Errors
- Clear Expo cache: `expo r -c`
- Reset Metro cache: `npx react-native start --reset-cache`
- Reinstall dependencies: `rm -rf node_modules && npm install`

### Navigation Issues
- Ensure all screens are properly imported
- Check navigation parameter types
- Verify screen names match navigation calls

### Image Issues
- Check image paths and extensions
- Verify storage bucket permissions
- Test with placeholder images first

### Database Connection
- Verify Supabase URL and key
- Check RLS policies
- Test database connection in Supabase dashboard

## 11. Deployment Preparation

### App Store (iOS)
1. Create App Store Connect account
2. Configure app metadata
3. Generate production build with EAS
4. Submit for review

### Google Play Store (Android)
1. Create Google Play Console account
2. Configure app listing
3. Generate signed APK/AAB with EAS
4. Submit for review

### Expo Updates (OTA)
```bash
# Publish update
eas update --branch production

# Configure auto-updates in app.json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/[project-id]"
    }
  }
}
```

## 12. Performance Optimization

### Image Optimization
- Use WebP format for better compression
- Implement lazy loading for product images
- Set up image caching strategies

### Data Optimization
- Implement pagination for large lists
- Use React Query for efficient caching
- Optimize database queries

### Bundle Optimization
- Enable Hermes for Android
- Use bundle splitting for larger apps
- Optimize asset loading

This completes the manual setup guide for the Expo mobile app. Follow these steps in order for a successful deployment.
