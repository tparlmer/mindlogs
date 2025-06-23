# Refactor Plan: Fragment Data Management

## Current State Analysis

**Current Architecture:**
- All patient data and thought fragments embedded as JavaScript objects in `index.html`
- ~6 patients with 3-4 fragments each = ~20 fragments total
- File size: ~2,100 lines, ~94KB
- Self-contained system with no external dependencies

**Growth Projections:**
- Target: 50+ fragments over 6 months
- 4 main characters + supporting characters
- Multiple fragment types (thought extracts, clinical notes, AI analysis)
- Estimated final size: ~5,000+ lines, ~200KB+

## Option 1: Keep Fragments in index.html ✅ RECOMMENDED

**Rationale: Emphasize Simplicity**
- Single file deployment - just copy `index.html` anywhere and it works
- No additional HTTP requests or loading complexity
- No need for error handling around missing files
- No async loading states or loading indicators needed
- Immediate availability of all content
- Perfect for prototyping and demo purposes

**When this becomes problematic:**
- File size >500KB (performance impact)
- >100 fragments (maintainability issues)
- Multiple content editors working simultaneously (merge conflicts)

**Mitigation strategies for growth:**
```javascript
// Organize data structure for maintainability
const PATIENTS = {
    clark: {
        info: { /* patient data */ },
        fragments: [ /* fragments */ ]
    },
    dave: { /* ... */ },
    amy: { /* ... */ },
    zoe: { /* ... */ }
};

// Or group by character for easier editing
const CLARK_FRAGMENTS = [ /* all Clark fragments */ ];
const DAVE_FRAGMENTS = [ /* all Dave fragments */ ];
// etc.
```

## Option 2: External JSON File (Not Recommended for Current Scale)

**Structure would be:**
```
fragments/
├── patients.json          # All patient data
└── index.html            # Interface only
```

**Implementation:**
```javascript
// In index.html
async function loadPatients() {
    const response = await fetch('fragments/patients.json');
    const patients = await response.json();
    return patients;
}
```

**Why not recommended now:**
- Adds complexity for marginal benefit
- Requires web server for local development (no file:// protocol)
- Need error handling for network failures
- Loading states and async complexity
- Multiple files to manage for deployment

## Recommended Approach: Gradual Refactor

### Phase 1: Organize Within index.html (Immediate)
```javascript
// Move from single patients array to organized structure
const FRAGMENT_DATA = {
    clark: {
        id: "P-1127-CD",
        name: "Clark DeGuerre",
        // ... patient info
        fragments: [
            // All Clark fragments here
        ]
    },
    dave: {
        // Dave's data
    },
    amy: {
        // Amy's data  
    },
    zoe: {
        // Zoe's data
    }
};

// Convert to patients array at runtime
const patients = Object.values(FRAGMENT_DATA);
```

### Phase 2: Content Management Tools (If Needed Later)
- Simple Node.js script to validate fragment JSON structure
- Fragment template generator for consistent formatting
- Bulk operations for updating metadata

### Phase 3: External Files (Only If >100 fragments)
- Move to separate JSON only when file size becomes unwieldy
- Implement proper loading states and error handling
- Add offline/caching strategy

## Implementation: Phase 1 Reorganization

**Benefits:**
- Easier to find specific character's fragments
- Cleaner separation of characters while staying in one file
- Simpler to add new fragments for specific characters
- Maintains simplicity while improving organization

**File structure within index.html:**
```javascript
// Character-organized data structure
const CHARACTERS = {
    clark: {
        patientInfo: { /* metadata */ },
        fragments: [ /* all Clark fragments */ ]
    }
    // etc.
};

// Runtime conversion for existing interface
const patients = Object.entries(CHARACTERS).map(([key, data]) => ({
    ...data.patientInfo,
    logs: data.fragments
}));
```

## Decision: Stay Single-File

**Recommendation: Keep all fragments in index.html**

**Reasoning:**
1. **Simplicity wins** - Single file deployment and management
2. **Current scale doesn't justify complexity** - 20-50 fragments manageable in one file
3. **Performance acceptable** - 200KB loads instantly on modern connections
4. **Easier content creation** - No need to manage multiple files or understand JSON structure
5. **Zero deployment complexity** - Works anywhere, no server required

**Future decision point:** 
Revisit when we hit 100+ fragments or file size >500KB, whichever comes first.

## Next Steps

1. **Immediate:** Reorganize existing fragments by character within index.html
2. **Content creation:** Use existing structure, add fragments directly to character sections
3. **Monitor:** Track file size and editing complexity
4. **Reassess:** When approaching 100 fragments or significant performance issues

The best architecture is the one that doesn't get in the way of creating great content.