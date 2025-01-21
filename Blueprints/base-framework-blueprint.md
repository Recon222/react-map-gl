# Base Framework Blueprint: Reusable Mapping Application Canvas

## Overview
This document outlines the architecture for a reusable mapping application framework designed to serve as a foundation for multiple specialized mapping applications. The goal is to create a clean, modular base that can be extended for specific use cases while maintaining consistency in core functionality and user experience.

## Motivation
### Current Challenges
- Legacy code accumulation across applications
- Inconsistent implementations of similar features
- Difficulty maintaining multiple codebases
- Redundant code between applications
- Inflexible marker/ID systems
- Inconsistent state management

### Solution
Create a clean, modular "canvas" framework that:
- Serves as a foundation for multiple mapping applications
- Provides consistent core functionality
- Maintains clean separation of concerns
- Uses modern best practices from the start
- Implements proper UUID-based systems
- Enables real-world testing at each stage

## Core Framework Components

### 1. Map Container System
```typescript
interface MapSystem {
  container: MapContainer;
  controls: MapControls;
  state: MapState;
  events: MapEvents;
}
```

Key Features:
- Clean Mapbox integration
- Configurable initial view
- Consistent event handling
- Flexible control system
- Performance optimized
- Type-safe interfaces

### 2. Panel System
```typescript
interface PanelSystem {
  panels: Map<string, Panel>;
  layout: PanelLayout;
  state: PanelState;
}

interface Panel {
  id: string;
  type: 'floating' | 'docked' | 'minimized';
  position: Position;
  size: Size;
  zIndex: number;
}
```

Features:
- Draggable panels
- Docking system
- Size management
- Position memory
- Z-index handling
- Edge snapping
- Panel stacking

### 3. State Management
```typescript
interface BaseState {
  map: MapState;
  panels: PanelState;
  markers: MarkerState;
  selection: SelectionState;
}
```

Features:
- Clean context structure
- Type-safe state
- Modular reducers
- Performance optimized
- Extensible design
- State persistence

### 4. Base Marker System
```typescript
interface BaseMarker {
  id: string;          // UUID v4
  type: string;        // Marker type identifier
  position: Position;  // Lat/Lng
  metadata: Record<string, unknown>;
  created: string;     // ISO timestamp
  updated: string;     // ISO timestamp
}
```

Features:
- UUID-based identification
- Type-safe marker system
- Extensible metadata
- Timestamp tracking
- Change tracking
- Position management

## Technical Specifications

### 1. Core Technologies
- React 18+
- TypeScript 5+
- Mapbox GL JS
- Tailwind CSS
- Vite

### 2. Key Design Patterns
- Composition over inheritance
- Dependency injection
- Observer pattern for events
- Strategy pattern for behaviors
- Factory pattern for markers
- Repository pattern for data

### 3. State Management Principles
- Single source of truth
- Immutable state updates
- Action-based mutations
- Type-safe contexts
- Modular state slices
- Optimized renders

### 4. Panel Management
```typescript
interface PanelManager {
  createPanel: (config: PanelConfig) => Panel;
  dockPanel: (id: string, position: DockPosition) => void;
  floatPanel: (id: string, position: Position) => void;
  minimizePanel: (id: string) => void;
  stackPanel: (id: string, targetId: string) => void;
}
```

### 5. Event System
```typescript
interface EventSystem {
  register: (event: string, handler: EventHandler) => void;
  unregister: (event: string, handler: EventHandler) => void;
  emit: (event: string, data: unknown) => void;
}
```

## Extension Points

### 1. Marker Extensions
- Custom marker types
- Specialized behaviors
- Custom rendering
- Type-specific interactions

### 2. Panel Extensions
- Custom panel types
- Specialized layouts
- Custom behaviors
- Content providers

### 3. Map Extensions
- Custom controls
- Layer management
- Custom interactions
- Visualization plugins

### 4. State Extensions
- Custom reducers
- State middleware
- Persistence strategies
- Sync mechanisms

## Development Phases

### Phase 1: Core Framework
1. Basic map integration
2. Panel system foundation
3. State management setup
4. Event system implementation

### Phase 2: Panel System
1. Draggable panels
2. Docking system
3. Panel management
4. Position memory

### Phase 3: Marker System
1. UUID implementation
2. Marker management
3. Selection system
4. Change tracking

### Phase 4: Testing & Optimization
1. Real-world testing
2. Performance optimization
3. Edge case handling
4. Documentation

## Best Practices

### UUID Implementation
- Use UUID v4 for all entities
- Maintain referential integrity
- Handle UUID collisions
- Implement proper validation

### State Management
- Keep state normalized
- Use proper typing
- Implement proper actions
- Maintain clean reducers

### Panel Management
- Handle window resizing
- Manage z-index properly
- Implement proper stacking
- Handle edge cases

### Performance
- Implement proper memoization
- Use virtual lists when needed
- Optimize re-renders
- Handle large datasets

## Future Considerations

### Scalability
- Support for large datasets
- Handle multiple map instances
- Support for complex layouts
- Handle heavy processing

### Extensions
- Plugin system
- Custom controls
- Visualization tools
- Data processors

### Integration
- API integration
- Service workers
- Offline support
- Real-time updates

## Testing Strategy

### Unit Testing
- Component testing
- State management
- Event handling
- UUID management

### Integration Testing
- Panel interactions
- Map integration
- State updates
- Event propagation

### Real-World Testing
- Performance testing
- User interactions
- Edge cases
- Error handling

## Next Steps
1. Set up basic project structure
2. Implement map container
3. Create panel system
4. Implement state management
5. Build marker system

This blueprint serves as a foundation for creating a clean, reusable mapping application framework that can be extended for various specific use cases while maintaining consistency and proper architecture.