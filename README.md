# Building a Modern Layout System with Chakra UI and React

This guide explains how to create a flexible, responsive layout system similar to the one used in this project. It covers file organization, styling with Chakra UI, performance optimization with React.memo, and best practices for maintainable code.

## Table of Contents

1. [Project Structure](#project-structure)
2. [Setting Up Chakra UI](#setting-up-chakra-ui)
3. [Creating the Layout System](#creating-the-layout-system)
4. [Styling with Chakra UI](#styling-with-chakra-ui)
5. [Using Theme Tokens](#using-theme-tokens)
6. [Performance Optimization with React.memo](#performance-optimization-with-reactmemo)
7. [Custom Hooks for Reusable Logic](#custom-hooks-for-reusable-logic)
8. [Best Practices](#best-practices)

## Project Structure

A well-organized project structure is essential for maintainability. Here's a recommended structure based on the current project:

```
src/
├── assets/           # Static assets (icons, images, global CSS)
├── atoms/            # Smallest UI components (buttons, inputs)
├── components/       # Reusable UI components
│   ├── ComponentName/
│   │   ├── index.tsx           # Component implementation
│   │   └── styles/             # Component styles
│   │       ├── index.ts        # Exports all styles
│   │       └── Component.ts    # Component-specific styles
├── constants/        # Application constants and configuration
├── containers/       # Container components that manage state
│   ├── LayoutContainer/
│   │   ├── index.tsx           # Container implementation
│   │   ├── components/         # Container-specific components
│   │   └── styles/             # Container styles
│   │       ├── index.ts        # Exports all styles
│   │       └── LayoutStyles.ts # Container-specific styles
│   ├── MapContainer/
│   │   ├── index.tsx
│   │   └── styles/
│   ├── SideBarContainer/
│   │   ├── index.tsx
│   │   └── styles/
│   └── ... other containers
├── context/          # React context providers
├── hooks/            # Custom React hooks
├── styled/           # Styled components and global styles
├── theming/          # Chakra UI theme customization
│   ├── components/           # Component-specific theme overrides
│   ├── globalCss.ts          # Global CSS styles
│   └── index.tsx             # Theme configuration
├── types/            # TypeScript type definitions
└── utils/            # Utility functions
```

## Setting Up Chakra UI

### Installation

```bash
npm install @penta-b/chakra-ui @emotion/react
```

Note: This project uses a custom version of Chakra UI (`@penta-b/chakra-ui`), but you can use the official package:

```bash
npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

### Theme Setup

Create a theme configuration file to customize Chakra UI:

```typescript
// src/theming/index.tsx
import { extendTheme } from '@chakra-ui/react';
import { globalCss } from './globalCss';

// Define your theme tokens
const theme = extendTheme({
  // Colors
  colors: {
    brand: {
      50: '#e6f7ff',
      100: '#b3e0ff',
      // ... other shades
      900: '#003366',
    },
  },
  
  // Typography
  fonts: {
    heading: '"Cairo", sans-serif',
    body: '"Cairo", sans-serif',
  },
  
  // Breakpoints for responsive design
  breakpoints: {
    sm: '30em',    // 480px
    md: '48em',    // 768px
    lg: '62em',    // 992px
    xl: '80em',    // 1280px
    '2xl': '96em', // 1536px
  },
  
  // Component-specific theming
  components: {
    Button: {
      // Custom variants
      variants: {
        solid: {
          bg: 'brand.500',
          color: 'white',
          _hover: {
            bg: 'brand.600',
          },
        },
      },
    },
  },
  
  // Global styles
  styles: {
    global: globalCss,
  },
});

export default theme;
```

### Global CSS

```typescript
// src/theming/globalCss.ts
export const globalCss = {
  'html, body': {
    height: '100%',
    margin: 0,
    padding: 0,
    fontFamily: 'body',
  },
  '#root': {
    height: '100%',
  },
  '*': {
    boxSizing: 'border-box',
  },
};
```

### Layout Constants in Theming

Integrate layout constants with your Chakra UI theme for better consistency. This approach leverages Chakra UI's breakpoint system and ensures your layout values are accessible throughout your application.

#### 1. Define Breakpoints in Theme

First, ensure your breakpoints are defined in your Chakra UI theme:

```typescript
// src/theming/index.tsx
import { extendTheme } from '@chakra-ui/react';

// Breakpoints for responsive design
const breakpoints = {
  sm: '30em',    // 480px
  md: '48em',    // 768px - Mobile breakpoint
  lg: '62em',    // 992px
  xl: '80em',    // 1280px
  '2xl': '96em', // 1536px
};

const theme = extendTheme({
  // Other theme properties...
  breakpoints,
});

export default theme;
```


#### 2. Using Theme Tokens

Instead of using external constants, define all design tokens directly in the Chakra UI theme:

```typescript
// src/theming/index.tsx
const system = createSystem(defaultConfig, {
  preflight: true,
  cssVarsPrefix: LOCALIZATION_NAMESPACE,
  globalCss,
  theme: {
    tokens: {
      colors,
      fonts,
      breakpoints,
      // Z-index scale
      zIndices: {
        base: 0,
        content: 1,
        overlay: 100,
        modal: 1000,
        tooltip: 1100,
        navbar: 100,
      },
      // Animation/transition tokens
      transition: {
        fast: '150ms',
        normal: '300ms',
        slow: '500ms',
        easeInOut: 'ease-in-out',
      },
      // Layout spacing tokens
      space: {
        navbarHeight: '50px',
        bottomToolbarHeight: '60px',
        desktopSidebarWidth: '512px',
      },
      // Border radius tokens
      radii: {
        desktop: '12px',
        mobile: '0px',
        card: '12px',
      },
    },
    // Semantic tokens for responsive/contextual values
    semanticTokens: {
      space: {
        'container-padding': {
          base: '0', // 0px mobile
          md: '2',   // 8px desktop
        },
      },
      sizes: {
        'sidebar-width': {
          base: '100%',
          lg: 'desktopSidebarWidth',
        },
      },
    },
    recipes
  },
});
```

## Using Theme Tokens

Chakra UI's theme token system provides a more maintainable and type-safe way to manage design values compared to external constants. Here's how to use theme tokens effectively:

### Direct Token References

Use string references to theme tokens in style objects:

```tsx
// Before (using constants)
const styles = {
  height: LAYOUT_CONSTANTS.NAVBAR_HEIGHT,
  zIndex: Z_INDEX.OVERLAY,
  borderRadius: LAYOUT_CONSTANTS.BORDER_RADIUS.DESKTOP,
};

// After (using theme tokens)
const styles = {
  height: "navbarHeight",
  zIndex: "overlay",
  borderRadius: "desktop",
};
```

### Semantic Tokens for Responsive Values

Use semantic tokens to handle responsive values with a single property:

```tsx
// Before (using responsive object)
<Box
  padding={{
    base: LAYOUT_CONSTANTS.SPACING.MOBILE_PADDING,
    md: LAYOUT_CONSTANTS.SPACING.DESKTOP_PADDING
  }}
/>

// After (using semantic token)
<Box padding="container-padding" />
```

### CSS Variables for Dynamic Values

Access theme tokens as CSS variables in string templates:

```tsx
// Before (using constants)
const transition = `all ${ANIMATION.NORMAL} ease-in-out`;

// After (using CSS variables)
const transition = "all var(--dispatch-layout-transition-normal) var(--dispatch-layout-transition-easeInOut)";
```

### Programmatic Access with useTheme

For cases where you need to access theme values in JavaScript:

```tsx
import { useTheme } from "@penta-b/chakra-ui";

function MyComponent() {
  const theme = useTheme();
  const navbarHeight = theme.space.navbarHeight;
  
  // Use the value in calculations
  const contentHeight = `calc(100vh - ${navbarHeight})`;
  
  return <Box height={contentHeight} />;
}
```

### Example Component Using Theme Tokens

```tsx
// src/containers/SideBarContainer/styles/SideBarContainer.ts
import { BoxProps } from "@penta-b/chakra-ui";

// Sidebar Container Styles
export const sidebarContainerStyles: BoxProps = {
  id: "sidebar-container",
  maxHeight: {
    base: `calc(100vh - var(--dispatch-layout-space-navbarHeight))`,
    lg: "95%"
  },
  width: "sidebar-width", // Using semantic token for responsive width
  overflowY: "auto",
  p: "container-padding", // Using semantic token for responsive padding
  role: "complementary",
  transition: "all var(--dispatch-layout-transition-normal) var(--dispatch-layout-transition-easeInOut)"
};
```

### Benefits of Using Theme Tokens

1. **Single Source of Truth**: All design values are defined in the theme
2. **Type Safety**: Better TypeScript autocomplete and validation
3. **CSS Custom Properties**: Automatic generation of CSS variables
4. **Responsive Behavior**: Simplified responsive token usage
5. **Consistency**: Standardized naming and usage patterns

For a complete migration guide, see the [THEME_MIGRATION_GUIDE.md](./THEME_MIGRATION_GUIDE.md) file.

#### 3. Using Breakpoints in Components

Use Chakra UI's responsive utilities with these breakpoints:

```typescript
import { Box } from '@chakra-ui/react';
import { LAYOUT_CONSTANTS } from '../constants';

// Responsive component example
const ResponsiveContainer = () => (
  <Box
    width="100%"
    padding={{
      base: LAYOUT_CONSTANTS.SPACING.MOBILE_PADDING,
      md: LAYOUT_CONSTANTS.SPACING.DESKTOP_PADDING
    }}
    borderRadius={{
      base: LAYOUT_CONSTANTS.BORDER_RADIUS.MOBILE,
      md: LAYOUT_CONSTANTS.BORDER_RADIUS.DESKTOP
    }}
  >
    Content
  </Box>
);
```

## Creating the Layout System

### Main Layout Container

```tsx
// src/containers/LayoutContainer/index.tsx
import React from "react";
import { Box } from "@chakra-ui/react";
import NavBar from "../../components/NavBar";
import { LAYOUT_CONSTANTS } from "../../constants";
import { MobileLayout, DesktopLayout } from "./components";

const LayoutContent: React.FC = () => {
  return (
    <Box
      height="100vh"
      width="full"
      pr={0}
      pl={{
        base: LAYOUT_CONSTANTS.SPACING.MOBILE_PADDING,
        md: LAYOUT_CONSTANTS.SPACING.DESKTOP_PADDING
      }}
      position="relative"
      m={0}
      overflow="hidden"
    >
      {/* Top NavBar */}
      <NavBar />

      {/* Main content area below navbar */}
      <Box display={{ base: 'block', md: 'none' }}>
        <MobileLayout />
      </Box>
      <Box display={{ base: 'none', md: 'block' }}>
        <DesktopLayout />
      </Box>
    </Box>
  );
};

/**
 * Main layout container component
 * Serves as the entry point for the layout system
 */
const LayoutContainer: React.FC = () => {
  return <LayoutContent />;
};

export default LayoutContainer;
```

### Desktop Layout

```tsx
// src/containers/LayoutContainer/components/DesktopLayout.tsx
import React from "react";
import { Grid, GridItem, Box } from "@chakra-ui/react";
import { useSidebar } from "../../../context/SidebarContext";
import MapContainer from "../../MapContainer";
import PluginsTriggerContainer from "../../PluginsTriggerContainer";
import MainPluginsTriggerContainer from "../../MainPluginsTriggerContainer";
import DesktopSidebarWrapper from "./DesktopSidebarWrapper";
import {
  getDesktopLayoutGridStyles,
  desktopPluginsTriggerContainerStyles,
  desktopPluginsTriggerBoxStyles,
  desktopMapContainerStyles,
  desktopMapBoxStyles
} from "./styles";
import { Plugin } from "../../../types/common";

interface DesktopLayoutProps {
  plugins: Plugin[];
}

/**
 * Desktop-specific layout component
 */
const DesktopLayoutComponent: React.FC<DesktopLayoutProps> = ({ plugins = [] }) => {
  const { isVisible } = useSidebar();
  const hasPlugins = plugins.length > 0;

  return (
    <Grid {...getDesktopLayoutGridStyles(hasPlugins, isVisible)}>
      {/* Plugins Trigger Container */}
      <GridItem {...desktopPluginsTriggerContainerStyles}>
        <Box {...desktopPluginsTriggerBoxStyles}>
          <PluginsTriggerContainer />
        </Box>
      </GridItem>

      {/* Sidebar Container - DesktopSidebarWrapper handles empty arrays */}
      <DesktopSidebarWrapper />

      {/* Map Container - always renders, takes remaining space */}
      <GridItem {...desktopMapContainerStyles}>
        <Box {...desktopMapBoxStyles}>
          <MapContainer />
          {/* Main Plugins Trigger Container positioned at the bottom */}
          <MainPluginsTriggerContainer />
        </Box>
      </GridItem>
    </Grid>
  );
};

export default DesktopLayoutComponent;
```

### Mobile Layout

```tsx
// src/containers/LayoutContainer/components/MobileLayout.tsx
import React from "react";
import { Box } from "@penta-b/chakra-ui";
import MapContainer from "../../MapContainer";
import MobileBottomContainer from "../../MobileBottomContainer";
import {
  mobileLayoutContainerStyles,
  mobileMapContainerWrapperStyles,
  mobileLayoutBottomToolbarStyles
} from "./styles";

/**
 * Mobile-specific layout component
 */
const MobileLayout: React.FC = () => {
  return (
    <Box {...mobileLayoutContainerStyles}>
      {/* Map Container */}
      <Box {...mobileMapContainerWrapperStyles}>
        <MapContainer />
      </Box>
      
      {/* Bottom Toolbar */}
      <Box {...mobileLayoutBottomToolbarStyles}>
        <MobileBottomContainer />
      </Box>
    </Box>
  );
};

export default MobileLayout;
```

### Mobile Bottom Container

```tsx
// src/containers/MobileBottomContainer/index.tsx
import React, { memo, useState, useEffect } from 'react';
import { connect } from 'react-redux';
import {
  Box,
  Drawer,
  VisuallyHidden,
} from "@penta-b/chakra-ui";
import { Plugin, AppState, LocalizedProps } from '../../types/common';
import MinimizeIcon from '../../assets/icon/minimize-icon';
import { useDragToClose } from '../../hooks';

interface MobileBottomContainerProps extends LocalizedProps {
  plugins?: Plugin[];
}

interface StateProps {
  plugins?: Plugin[];
}

/**
 * MobileBottomContainer component renders plugins in a bottom drawer
 * Optimized with React.memo to prevent unnecessary re-renders
 */
const MobileBottomContainer: React.FC<MobileBottomContainerProps> = memo(({ plugins = [] }) => {
  const [isDrawerOpen, setIsDrawerOpen] = useState(false);
  const [isMinimized, setIsMinimized] = useState(false);
  
  const { handlePointerDown, handlePointerMove, handlePointerUp } = useDragToClose({
    onClose: () => setIsMinimized(true),
    threshold: 40,
  });
  
  // Use useEffect to update drawer state based on plugins
  useEffect(() => {
    if (plugins && plugins.length > 0) {
      setIsDrawerOpen(true);
      setIsMinimized(false);
    }
  }, [plugins]);
  
  // Handle minimize/maximize
  const handleMinimize = () => {
    setIsMinimized(true);
  };
  
  const handleMaximize = () => {
    setIsMinimized(false);
  };
  
  // Early return if no plugins to render
  if (!plugins || plugins.length === 0) {
    return null;
  }

  return (
    <>
      {isMinimized ? (
        // Minimized state - show only a handle to drag up
        <Box
          position="fixed"
          bottom="0"
          left="0"
          right="0"
          height="40px"
          bg="white"
          boxShadow="0 -2px 10px rgba(0,0,0,0.1)"
          zIndex={900}
          display="flex"
          alignItems="center"
          justifyContent="center"
          onClick={handleMaximize}
          style={{ touchAction: "none" }}
          onPointerDown={handlePointerDown}
          onPointerMove={handlePointerMove}
          onPointerUp={handlePointerUp}
        >
          <Box
            width="48px"
            height="5px"
            bg="#EEEEEE"
            borderRadius="full"
            mb="2"
          />
          <Box position="absolute" right="12px">
            <Box
              as="button"
              aria-label="Maximize"
              onClick={handleMaximize}
              display="flex"
              alignItems="center"
              justifyContent="center"
              p="2"
            >
              <Box transform="rotate(180deg)">
                <MinimizeIcon width={20} height={20} />
              </Box>
            </Box>
          </Box>
        </Box>
      ) : (
        // Full drawer state
        <Drawer.Root
          open={isDrawerOpen}
          onOpenChange={(details: { open: boolean }) =>
            setIsDrawerOpen(details.open)
          }
          placement="bottom"
          modal={false}
          preventScroll={false}
        >
          <Drawer.Positioner
            paddingBottom="60px"
            zIndex={900}
            pointerEvents="none"
          >
            <Drawer.Content
              boxShadow="none"
              pointerEvents="auto"
              portalled={false}
              zIndex={900}
            >
              <Drawer.Header
                position="relative"
                pt="6"
                style={{ touchAction: "none" }}
                onPointerDown={handlePointerDown}
                onPointerMove={handlePointerMove}
                onPointerUp={handlePointerUp}
              >
                <Drawer.Title>
                  <VisuallyHidden>More Plugins</VisuallyHidden>
                </Drawer.Title>
                <Drawer.CloseTrigger
                  asChild
                  position="absolute"
                  top="3"
                  left="50%"
                  transform="translateX(-50%)"
                  aria-label="Close"
                >
                  <Box
                    width="48px"
                    height="5px"
                    bg="#EEEEEE"
                    borderRadius="full"
                  />
                </Drawer.CloseTrigger>
                <Box 
                  position="absolute" 
                  top="2" 
                  left="2"
                  as="button"
                  onClick={handleMinimize}
                  aria-label="Minimize"
                >
                  <MinimizeIcon width={24} height={24} />
                </Box>
              </Drawer.Header>
              <Drawer.Body>
                {plugins.map((plugin) => {
                  const Component = plugin.Component;
                  return (
                    <Box 
                      key={plugin.id}
                    >
                      <Component {...plugin.props} />
                    </Box>
                  );
                })}
              </Drawer.Body>
            </Drawer.Content>
          </Drawer.Positioner>
        </Drawer.Root>
      )}
    </>
  );
});

// Display name for debugging
MobileBottomContainer.displayName = 'MobileBottomContainer';

const mapStateToProps = (state: AppState): StateProps => ({
  plugins: selectorsRegistry.getSelector('selectPanelComponents', state, 'mobile-bottom-container')
});

export default connect(mapStateToProps)(MobileBottomContainer);
```

## Styling with Chakra UI

### Component Style Files Organization

Organize styles in dedicated files for better maintainability:

```
components/
└── ComponentName/
    ├── index.tsx           # Component implementation
    └── styles/             # Component styles folder
        ├── index.ts        # Exports all styles
        └── Component.ts    # Component-specific styles
```

#### Style Index File

```typescript
// src/components/ComponentName/styles/index.ts
export * from './Component';
export * from './SubComponent';
// Export all style files for easy imports
```

#### Component Styles

Create separate style files for each component:

```typescript
// src/containers/MapContainer/styles/MapContainer.ts
import { BoxProps, ButtonProps } from "@chakra-ui/react";
import { LAYOUT_CONSTANTS } from "../../../constants";

// Main Container Styles
export const getMapContainerStyles = (zIndex: number): BoxProps => ({
  id: "map-container",
  width: "100%",
  height: "100%",
  position: "relative",
  zIndex: zIndex,
  display: "block",
  overflow: "hidden"
});

// Show Sidebar Button Styles
export const showSidebarButtonStyles: ButtonProps = {
  id: "show-sidebar-button",
  position: "absolute",
  top: "15px",
  right: "15px",
  size: "sm",
  borderRadius: "full",
  width: "48px",
  height: "48px",
  backgroundColor: "white",
  boxShadow: "md",
  "aria-label": "Show sidebar"
};

// Export all styles for this component
export const mapContainerStyles = {
  container: getMapContainerStyles,
  showSidebarButton: showSidebarButtonStyles,
  // Add more styles as needed
};
```

### Responsive Styles

Use Chakra UI's responsive syntax for different screen sizes:

```typescript
// src/containers/SideBarContainer/styles/SideBarContainer.ts
import { BoxProps } from "@chakra-ui/react";
import { LAYOUT_CONSTANTS, COMPONENT_CONSTANTS } from "../../../constants";

// Sidebar Container Styles
export const sidebarContainerStyles: BoxProps = {
  id: "sidebar-container",
  maxHeight: {
    base: COMPONENT_CONSTANTS.SIDEBAR.MAX_HEIGHT_MOBILE,
    lg: COMPONENT_CONSTANTS.SIDEBAR.MAX_HEIGHT_DESKTOP
  },
  width: { base: "100%", lg: LAYOUT_CONSTANTS.DESKTOP_SIDEBAR_WIDTH },
  overflowY: "auto",
  p: {
    base: LAYOUT_CONSTANTS.SPACING.MOBILE_PADDING,
    lg: LAYOUT_CONSTANTS.SPACING.DESKTOP_PADDING
  },
  role: "complementary",
  transition: LAYOUT_CONSTANTS.TRANSITIONS.DEFAULT
};
```

### Dynamic Styles

Create functions for styles that depend on props or state:

```typescript
// src/containers/PluginsContentContainer/styles/ContentContainer.ts
import { BoxProps } from "@chakra-ui/react";

// Content Container Styles Function
export const getContentContainerStyles = (isResizing: boolean, width: number): BoxProps => ({
  width: `${width}px`,
  height: "100%",
  overflow: "hidden",
  display: "flex",
  flexDirection: "column",
  bg: "whiteAlpha.900",
  border: "1px solid",
  boxShadow: "0 10px 15px -3px rgba(0,0,0,0.1), 0 4px 6px -4px rgba(0,0,0,0.1)",
  borderRadius: "lg",
  transition: isResizing ? "none" : "width 0.2s ease"
});
```

### Style Composition and Reuse

Create base styles that can be extended and reused across components:

```typescript```
// src/containers/shared/styles/baseStyles.ts
import { BoxProps, FlexProps } from "@penta-b/chakra-ui";
import { LAYOUT_CONSTANTS } from "../../../constants";
```
// Base container styles that can be reused
export const baseContainerStyles: BoxProps = {
  width: "100%",
  height: "100%",
  overflow: "hidden",
  position: "relative",
  transition: LAYOUT_CONSTANTS.TRANSITIONS.DEFAULT
};

// Base card styles for consistent card appearance
export const baseCardStyles: BoxProps = {
  bg: "white",
  borderRadius: {
    base: LAYOUT_CONSTANTS.BORDER_RADIUS.MOBILE,
    md: LAYOUT_CONSTANTS.BORDER_RADIUS.DESKTOP
  },
  boxShadow: "md",
  overflow: "hidden",
  p: 4
};

// Base flex container for layouts
export const baseFlexContainerStyles: FlexProps = {
  display: "flex",
  width: "100%",
  height: "100%"
};
```

Then use these base styles in your component styles:

```typescript```
// src/containers/PluginContainer/styles/PluginContainer.ts
import { BoxProps } from "@penta-b/chakra-ui";
import { baseContainerStyles, baseCardStyles } from "../../shared/styles/baseStyles";
```
// Extend base styles for specific component
export const pluginContainerStyles: BoxProps = {
  ...baseContainerStyles,
  maxWidth: "800px",
  margin: "0 auto"
};

// Plugin card styles
export const pluginCardStyles: BoxProps = {
  ...baseCardStyles,
  mb: 4,
  _last: {
    mb: 0
  }
};
```

## Performance Optimization with React.memo

### Using React.memo

Wrap components with React.memo to prevent unnecessary re-renders:

```tsx
// src/containers/MapSearchBar/index.tsx
import React, { memo } from 'react';
import { connect } from 'react-redux';
import { Box } from '@penta-b/chakra-ui';
import { Plugin, AppState, LocalizedProps } from '../../types/common';
import {
  mapSearchBarContainerStyles,
  getMapSearchPluginItemStyles
} from './styles';

interface MapSearchBarProps extends LocalizedProps {
  plugins?: Plugin[];
}

interface StateProps {
  plugins?: Plugin[];
}

/**
 * MapSearchBar component renders search-related plugins in a fixed position overlay
 * Optimized with React.memo to prevent unnecessary re-renders
 */
const MapSearchBar: React.FC<MapSearchBarProps> = memo(({ plugins = [] }) => {
  // Early return if no plugins to render
  if (!plugins || plugins.length === 0) {
    return null;
  }
  
  return (
    <Box {...mapSearchBarContainerStyles}>
      {plugins.map((plugin) => {
        const Component = plugin.Component;
        return (
          <Box 
            key={plugin.id}
            {...getMapSearchPluginItemStyles(plugin.id)}
          >
            <Component {...plugin.props} />
          </Box>
        );
      })}
    </Box>
  );
});

// Display name for debugging
MapSearchBar.displayName = 'MapSearchBar';

const mapStateToProps = (state: AppState): StateProps => ({
  plugins: selectorsRegistry.getSelector('selectPanelComponents', state, 'map-search-bar')
});

export default connect(mapStateToProps)(MapSearchBar);
```

### Using useCallback and useMemo

Use useCallback for functions and useMemo for computed values to prevent unnecessary recreations:

```tsx
// src/containers/SideBarContainer/index.tsx
import React, { memo, useCallback, useEffect } from 'react';
import { connect } from 'react-redux';
import { Box } from '@penta-b/chakra-ui';
import { Plugin, AppState, LocalizedProps } from '../../types/common';
import { useSidebarPluginContainer } from '../../hooks';
import { sidebarContainerStyles } from './styles';

interface StateProps {
  plugins?: Plugin[];
}

interface DispatchProps {
  removeComponent: (id: string) => void;
}

type SideBarContainerProps = StateProps & DispatchProps & LocalizedProps;

/**
 * Container component for sidebar plugins
 * Renders a list of plugins with headers and separators
 * Optimized with React.memo and useCallback to prevent unnecessary re-renders
 */
const SideBarContainer: React.FC<SideBarContainerProps> = memo(
  ({ plugins, removeComponent, t }) => {

    const { hasPlugins, filteredPlugins } = useSidebarPluginContainer(plugins);
    const lastPlugin = hasPlugins
      ? filteredPlugins[filteredPlugins.length - 1]
      : undefined;

    // Memoize the remove handler to prevent unnecessary re-renders
    const handleRemoveComponent = useCallback(
      (uid: string) => {
        removeComponent(uid);
      },
      [removeComponent]
    );

    // Remove all plugins except the last one (the most recently activated)
    useEffect(() => {
      if (!hasPlugins || filteredPlugins.length <= 1) return;
      const pluginsToRemove = filteredPlugins.slice(0, -1);
      pluginsToRemove.forEach((p) => {
        // uses _uid which is what removeComponent expects in the rest of the app
        if (p?.props?._uid) {
          removeComponent(p.props._uid);
        }
      });
    }, [hasPlugins, filteredPlugins, removeComponent]);

    // Early return if no plugins to render
    if (!hasPlugins || !lastPlugin) {
      return null;
    }

    return (
      <Box {...sidebarContainerStyles}>
        {/* Render the last plugin */}
        {lastPlugin.Component && (
          <lastPlugin.Component {...lastPlugin.props} />
        )}
      </Box>
    );
  }
);

// Display name for debugging
SideBarContainer.displayName = 'SideBarContainer';

const mapStateToProps = (state: AppState): StateProps => ({
  plugins: selectorsRegistry.getSelector('selectPanelComponents', state, 'side-bar-container')
});

const mapDispatchToProps: DispatchProps = {
  removeComponent: actionsRegistry.getAction('removeComponent')
};

export default connect(mapStateToProps, mapDispatchToProps)(SideBarContainer);
```

## Custom Hooks for Reusable Logic

### Mobile Detection Hook

```typescript
// src/hooks/use-mobile.ts
import { useBreakpointValue } from '@penta-b/chakra-ui';
```
/**
 * Custom hook to detect if the current viewport is mobile size
 * Uses Chakra UI's useBreakpointValue for consistent responsive behavior
 * 
 * @returns {boolean} True if the current viewport is mobile size (below md breakpoint)
 */
const useIsMobile = (): boolean => {
  // Use Chakra UI's breakpoint system: mobile (base) = true, md and above = false
  // This aligns with Chakra UI's default md breakpoint of 768px
  // Provide fallback value to prevent undefined during SSR/initial render
  const isMobile = useBreakpointValue<boolean>({
    base: true,
    md: false
  }) ?? false;
  
  // Ensure boolean return type
  return !!isMobile;
};

export default useIsMobile;
```

This approach leverages Chakra UI's built-in breakpoint system rather than using media queries directly, ensuring consistency with the theme breakpoints defined in your theming configuration.

### Creating Custom Hooks

```typescript
// src/hooks/use-drag-to-close.ts
import { useRef } from 'react';

interface UseDragToCloseOptions {
  onClose: () => void;
  threshold?: number;
}

interface UseDragToCloseReturn {
  handlePointerDown: (e: React.PointerEvent<HTMLElement>) => void;
  handlePointerMove: (e: React.PointerEvent<HTMLElement>) => void;
  handlePointerUp: (e: React.PointerEvent<HTMLElement>) => void;
}

export const useDragToClose = ({
  onClose,
  threshold = 40,
}: UseDragToCloseOptions): UseDragToCloseReturn => {
  const dragStartY = useRef<number | null>(null);
  const isDragging = useRef<boolean>(false);
  const hasClosedOnDrag = useRef<boolean>(false);

  const handlePointerDown = (e: React.PointerEvent<HTMLElement>): void => {
    dragStartY.current = e.clientY;
    isDragging.current = false;
    hasClosedOnDrag.current = false;
  };

  const handlePointerMove = (e: React.PointerEvent<HTMLElement>): void => {
    if (dragStartY.current == null) return;
    
    const deltaY = e.clientY - dragStartY.current;
    
    // Start dragging after a small threshold to not interfere with taps/clicks
    if (!isDragging.current && Math.abs(deltaY) > 5) {
      isDragging.current = true;
      try {
        (e.currentTarget as HTMLElement).setPointerCapture(e.pointerId);
      } catch {}
    }
    
    if (deltaY > threshold && !hasClosedOnDrag.current) {
      onClose();
      hasClosedOnDrag.current = true;
    }
  };

  const handlePointerUp = (e: React.PointerEvent<HTMLElement>): void => {
    if (isDragging.current) {
      try {
        (e.currentTarget as HTMLElement).releasePointerCapture(e.pointerId);
      } catch {}
    }
    
    dragStartY.current = null;
    isDragging.current = false;
  };

  return {
    handlePointerDown,
    handlePointerMove,
    handlePointerUp,
  };
};
```

### Plugin Container Hook

```typescript
// src/hooks/use-plugin-container.ts
import { useMemo } from 'react';
import { Plugin } from '../types/common';

interface UsePluginContainerProps {
  plugins?: Plugin[];
  maxPlugins?: number;
  filterFn?: (plugin: Plugin) => boolean;
}

export const usePluginContainer = ({
  plugins = [],
  maxPlugins = Infinity,
  filterFn = () => true,
}: UsePluginContainerProps) => {
  // Memoize filtered plugins to prevent unnecessary recalculations
  const filteredPlugins = useMemo(() => {
    if (!plugins || plugins.length === 0) return [];
    
    return plugins
      .filter(plugin => {
        // Skip plugins without a Component
        if (!plugin.Component) return false;
        
        // Apply custom filter function
        return filterFn(plugin);
      })
      .slice(0, maxPlugins); // Limit number of plugins
  }, [plugins, maxPlugins, filterFn]);
  
  // Memoize hasPlugins flag
  const hasPlugins = useMemo(() => {
    return filteredPlugins.length > 0;
  }, [filteredPlugins]);
  
  return {
    filteredPlugins,
    hasPlugins,
  };
};

/**
 * Hook specifically for sidebar plugin containers
 * Provides additional sidebar-specific functionality
 */
export const useSidebarPluginContainer = (plugins?: Plugin[]) => {
  const containerData = usePluginContainer({ plugins });
  
  // Memoize plugin IDs for efficient comparison
  const pluginIds = useMemo(() => {
    return containerData.filteredPlugins.map(plugin => plugin.id);
  }, [containerData.filteredPlugins]);
  
  // Check if a specific plugin is active
  const isPluginActive = useMemo(() => {
    return (pluginId: string) => pluginIds.includes(pluginId);
  }, [pluginIds]);
  
  return {
    ...containerData,
    pluginIds,
    isPluginActive
  };
};
```

### Context for Shared State

```typescript
// src/context/SidebarContext.tsx
import React, { createContext, useState, useContext, ReactNode, useCallback, useMemo } from 'react';

interface SidebarContextType {
  isVisible: boolean;
  toggle: () => void;
  show: () => void;
  hide: () => void;
}

const SidebarContext = createContext<SidebarContextType | undefined>(undefined);

interface SidebarProviderProps {
  children: ReactNode;
}

export const SidebarProvider: React.FC<SidebarProviderProps> = ({ children }) => {
  const [isVisible, setIsVisible] = useState(false);

  // New consolidated API - memoized with useCallback
  const toggle = useCallback(() => {
    setIsVisible(prev => !prev);
  }, []);

  const show = useCallback(() => {
    setIsVisible(true);
  }, []);

  const hide = useCallback(() => {
    setIsVisible(false);
  }, []);

  // Memoize the context value to prevent unnecessary re-renders
  const contextValue = useMemo(() => ({
    isVisible,
    toggle,
    show,
    hide
  }), [isVisible, toggle, show, hide]);

  return (
    <SidebarContext.Provider value={contextValue}>
      {children}
    </SidebarContext.Provider>
  );
};

export const useSidebar = (): SidebarContextType => {
  const context = useContext(SidebarContext);
  if (context === undefined) {
    throw new Error('useSidebar must be used within a SidebarProvider');
  }
  return context;
};
```

## Best Practices

### Type Safety

1. **Define Proper Interfaces**
   - Create interfaces for all component props
   - Use TypeScript for type checking
   - Avoid using `any` type

```typescript
// src/types/common.ts
export interface Plugin {
  id: string;
  Component: React.ComponentType<any>;
  props: any;
}

export interface AppState {
  systemReducer?: any;
  // Add other reducers as needed
}

export interface LocalizedProps {
  t: (key: string) => string;
  // Add other localization props as needed
}
```

### Performance Optimization

1. **Use React.memo for Pure Components**
   - Wrap components with React.memo to prevent unnecessary re-renders
   - Add displayName for better debugging

2. **Use useCallback for Event Handlers**
   - Memoize event handlers with useCallback
   - Pass stable references to child components

3. **Use useMemo for Expensive Calculations**
   - Memoize expensive calculations with useMemo
   - Avoid recalculating values on every render

4. **Early Returns**
   - Use early returns to avoid rendering unnecessary components
   - Check for null or empty arrays before rendering

### Code Organization

1. **Separate Styles from Components**
   - Keep styles in separate files
   - Use consistent naming conventions

2. **Use Constants for Magic Values**
   - Define constants for colors, sizes, and other values
   - Use semantic naming for constants

3. **Create Reusable Hooks**
   - Extract common logic into custom hooks
   - Keep hooks focused on a single responsibility

4. **Use Barrel Exports**
   - Create index files for cleaner imports
   - Export related components together

### Accessibility

1. **Use Semantic HTML**
   - Use appropriate HTML elements
   - Add ARIA attributes for better accessibility

2. **Keyboard Navigation**
   - Ensure components are keyboard accessible
   - Add focus management for modals and drawers

3. **Screen Reader Support**
   - Add descriptive labels for screen readers
   - Use VisuallyHidden component for screen reader text

### Error Handling

1. **Use Error Boundaries**
   - Wrap components with error boundaries
   - Provide fallback UI for errors

2. **Graceful Degradation**
   - Handle missing data gracefully
   - Provide default values for props

### Documentation

1. **Add JSDoc Comments**
   - Document component purpose and usage
   - Document props and return values

2. **Use Descriptive Variable Names**
   - Use clear and descriptive names
   - Avoid abbreviations and acronyms

### Testing

1. **Write Unit Tests**
   - Test components in isolation
   - Mock dependencies for testing

2. **Test Edge Cases**
   - Test with empty arrays and null values
   - Test error handling

## Responsive Design Best Practices

### Using Chakra UI's Responsive Props

Chakra UI provides a powerful responsive props system that allows you to define different values for different breakpoints:

```tsx
// Example of responsive props in a component
<Box
  width={{ base: "100%", md: "50%", lg: "33%" }}
  padding={{ base: 2, md: 4, lg: 6 }}
  display={{ base: "block", md: "flex" }}
>
  Content that adapts to different screen sizes
</Box>
```

### Responsive Style Objects

When defining style objects, use the same responsive syntax:

```typescript
// src/containers/SideBarContainer/styles/SideBarContainer.ts
import { BoxProps } from "@penta-b/chakra-ui";
import { LAYOUT_CONSTANTS } from "../../../constants";

// Sidebar Container Styles
export const sidebarContainerStyles: BoxProps = {
  id: "sidebar-container",
  maxHeight: {
    base: LAYOUT_CONSTANTS.COMPONENT_CONSTANTS.SIDEBAR.MAX_HEIGHT_MOBILE,
    lg: LAYOUT_CONSTANTS.COMPONENT_CONSTANTS.SIDEBAR.MAX_HEIGHT_DESKTOP
  },
  width: { base: "100%", lg: LAYOUT_CONSTANTS.DESKTOP_SIDEBAR_WIDTH },
  overflowY: "auto",
  p: {
    base: LAYOUT_CONSTANTS.SPACING.MOBILE_PADDING,
    lg: LAYOUT_CONSTANTS.SPACING.DESKTOP_PADDING
  },
  role: "complementary",
  transition: LAYOUT_CONSTANTS.TRANSITIONS.DEFAULT
};
```

### Conditional Rendering for Different Viewports

Use Chakra UI's responsive display property for conditional rendering:

```tsx
// src/containers/LayoutContainer/index.tsx
<Box display={{ base: 'block', md: 'none' }}>
  <MobileLayout />
</Box>
<Box display={{ base: 'none', md: 'block' }}>
  <DesktopLayout />
</Box>
```

### Responsive Grid Layouts

Leverage Chakra UI's Grid component for responsive layouts:

```tsx
// Example of responsive grid layout
<Grid
  templateColumns={{ 
    base: "1fr", 
    md: "auto 1fr", 
    lg: "auto auto 1fr" 
  }}
  gap={{ base: 2, md: 4, lg: 6 }}
>
  <GridItem>Sidebar</GridItem>
  <GridItem>Content</GridItem>
</Grid>
```

### Conclusion

By following these best practices and using the provided examples, you can create a flexible, responsive layout system with Chakra UI and React. The combination of proper file organization, type safety, performance optimization, and consistent use of Chakra UI's responsive system will result in a maintainable and scalable layout that works well across different devices and screen sizes.
