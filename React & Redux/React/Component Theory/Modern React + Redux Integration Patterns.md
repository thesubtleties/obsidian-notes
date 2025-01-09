*Speaker Platform Example*

## 1. Project Structure & Philosophy

```typescript
src/
  components/              // Global reusable components
    ui/                   
      Button/
      Card/
      Input/
    layout/               
      Grid/
      Container/
    feedback/             
      LoadingSpinner/
      ErrorMessage/
  features/               // Feature-based modules
    speakers/
      api/
      components/        // Presentational
      containers/        // Redux connection points
      hooks/            // Redux + Logic hooks
      store/            // Redux slice + selectors
```

**What:** Organized structure separating global reusable components from feature-specific code
**Why:** 
- Clear boundaries between reusable and feature-specific code
- Predictable places for Redux integration
- Easy to find connection points
- Maintains separation of concerns

## 2. Component Hierarchy & Redux Integration Points

### Page Level (Highest Level)
```tsx
// pages/SpeakersPage.tsx
const SpeakersPage = () => {
  // 🔑 High-level Redux connection point
  const { eventId } = useParams();
  const dispatch = useDispatch();
  
  // ✅ OK to connect here because:
  // - Multiple child components need this data
  // - Natural data boundary
  // - Reduces prop drilling
  const event = useSelector(selectEventDetails);

  useEffect(() => {
    dispatch(fetchEventData(eventId));
  }, [eventId]);

  return (
    <PageLayout>
      <EventHeader event={event} /> {/* Props drill is OK here */}
      <SpeakerListContainer /> {/* Handles own Redux connection */}
      <ScheduleContainer />   {/* Handles own Redux connection */}
    </PageLayout>
  );
};
```
**What:** Top-level page component that establishes main data requirements
**Why:**
- Creates clear data boundaries
- Prevents duplicate data fetching
- Manages high-level state
- Coordinates feature components

### Feature Container Level (Mid Level)
```tsx
// features/speakers/containers/SpeakerListContainer.tsx
const SpeakerListContainer = () => {
  // 🔑 Feature-specific Redux connection
  const {
    speakers,
    isLoading,
    error,
    handleUpdateSpeaker
  } = useSpeakers(); // Custom hook manages Redux

  // ✅ Connect here because:
  // - Feature-specific data needs
  // - Manages own slice of state
  // - Encapsulates Redux logic
  
  return (
    <Container>
      <SpeakerList
        speakers={speakers}
        onUpdate={handleUpdateSpeaker}
      />
    </Container>
  );
};
```
**What:** Container component that manages feature-specific Redux state
**Why:**
- Isolates Redux logic from presentation
- Provides clean API for feature components
- Manages loading/error states
- Single responsibility principle

### Component Level (Lowest Level)
```tsx
// features/speakers/components/SpeakerCard.tsx
// 🚫 No Redux connection here
const SpeakerCard = ({
  speaker,
  onUpdate
}: SpeakerCardProps) => (
  <Card>
    <Avatar src={speaker.image} />
    <Text>{speaker.name}</Text>
    <Button onClick={() => onUpdate(speaker.id)}>
      Update
    </Button>
  </Card>
);
```
**What:** Pure presentational component with no Redux knowledge
**Why:**
- Maximizes reusability
- Easier to test
- Clearer responsibility
- More maintainable

## 3. Global Components (Never Redux Connected)

```tsx
// components/ui/Button/Button.tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

const Button = ({
  variant = 'primary',
  size = 'md',
  isLoading,
  children,
  ...props
}: ButtonProps) => (
  <button 
    className={`btn-${variant} btn-${size}`}
    disabled={isLoading}
    {...props}
  >
    {isLoading ? <LoadingSpinner /> : children}
  </button>
);
```
**What:** Reusable UI component with no Redux dependencies
**Why:**
- Maximum reusability across features
- No business logic dependencies
- Clear, predictable behavior
- Easy to maintain and test

## 4. Redux Integration Strategy

### Custom Hooks Pattern
```tsx
// features/speakers/hooks/useSpeakers.ts
const useSpeakers = (filters?: SpeakerFilters) => {
  const dispatch = useDispatch();
  
  // 🔑 Memoized selector with parameters
  const speakers = useSelector(state => 
    selectFilteredSpeakers(state, filters)
  );

  // 🔑 Memoized action creators
  const actions = useMemo(() => ({
    updateSpeaker: (id: string, data: Partial<Speaker>) => 
      dispatch(speakerUpdated({ id, ...data })),
    deleteSpeaker: (id: string) => 
      dispatch(speakerDeleted(id))
  }), [dispatch]);

  return {
    speakers,
    ...actions
  };
};
```
**What:** Custom hook encapsulating Redux logic
**Why:**
- Reusable Redux logic
- Encapsulated state management
- Clean component integration
- Consistent data access pattern

