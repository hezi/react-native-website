---
id: migration-overview
title: Migration Guides
---

# Migration Guides

These guides will help you migrate your existing native modules and components from the legacy architecture to the new architecture (Turbo Modules and Fabric).

## Available Migration Guides

### Native Modules Migration

Learn how to migrate your existing native modules to Turbo Modules:

- **[Android Native Modules Migration](./migration/native-modules-migration-android)** - Step-by-step guide for migrating Android native modules
- **[iOS Native Modules Migration](./migration/native-modules-migration-ios)** - Step-by-step guide for migrating iOS native modules

### Native Components Migration

Learn how to migrate your existing native UI components to Fabric components:

- **[Android Native Components Migration](./migration/native-components-migration-android)** - Step-by-step guide for migrating Android native components
- **[iOS Native Components Migration](./migration/native-components-migration-ios)** - Step-by-step guide for migrating iOS native components

## When to Use These Guides

Use these migration guides when you:

- Have existing native modules or components built with the legacy architecture
- Want to adopt the new architecture but need to maintain backward compatibility
- Need a step-by-step approach to migrate your code incrementally
- Want to understand the differences between legacy and new architecture implementations

## Prerequisites

Before starting the migration, ensure you have:

1. **React Native 0.76 or higher** - The new architecture requires at least React Native 0.76
2. **New Architecture enabled** - Follow the [New Architecture setup guide](/architecture/landing-page#should-i-use-the-new-architecture-today) to enable it in your app
3. **Basic understanding** - Familiarity with the current native module/component APIs
4. **Development environment** - Properly configured development environment for Android/iOS development

## Migration Overview

The migration process typically involves:

1. **Creating TypeScript/Flow specifications** - Define the interface for your module/component
2. **Updating the native implementation** - Modify your Java/Kotlin or Objective-C/Swift code
3. **Adjusting the JavaScript interface** - Update how your JavaScript code interacts with the native side
4. **Testing and validation** - Ensure everything works correctly with both architectures

## Need Help?

If you encounter issues during migration:

- Check the [New Architecture Troubleshooting guide](/docs/new-architecture-troubleshooting) (Coming soon)
- Review the [New Architecture documentation](../../architecture/landing-page/)
- Ask questions in the [React Native Community](https://reactnative.dev/community/communities)

