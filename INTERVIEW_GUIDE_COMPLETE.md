# NoteFlow AI - Complete Interview Guide

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Component Workflows](#component-workflows)
4. [Key Features Explained](#key-features-explained)
5. [Interview Q&A](#interview-qa)
6. [Quick Reference](#quick-reference)

---

## Project Overview

**NoteFlow AI** is a Next.js-based AI-powered note-taking app with:
- Rich text editing (WYSIWYG)
- AI features (summarize, translate, generate tags via Groq API)
- Encryption (CryptoJS AES-256)
- Glossary highlighting
- Grammar checking
- Mobile responsive design
- Dark mode support (framework setup, no toggle yet)

**Tech Stack:**
- React 19.2 + Next.js 16.0.3
- Tailwind CSS v4.1.9
- React Context API (state management)
- localStorage (persistence)
- CryptoJS (encryption)
- Groq API llama-3.1-8b-instant (AI)
- Radix UI (components)

---

## Architecture

### Data Flow
```
User Input → React Component → Context API → localStorage → Display
     ↓              ↓                ↓            ↓
  Button        RichTextEditor    NotesContext   JSON
  Type          Editor            updateNote()   File
```

### Component Hierarchy
```
page.jsx (root)
  ├─ Header (app branding, note count, theme toggle)
  ├─ NotesList (sidebar - note management)
  └─ Editor (main area - editing interface)
      ├─ RichTextEditor (text editing)
      │   ├─ Toolbar (formatting)
      │   ├─ GrammarChecker (error detection)
      │   └─ GlossaryPanel (term definitions)
      ├─ AIPanel (modal - AI features)
      ├─ EncryptionModal (modal - encrypt/decrypt)
      └─ PasswordPrompt (modal - enter password)
```

### State Management (Context)
```
NotesContext
  ├─ notes: []
  ├─ createNote()
  ├─ updateNote()
  ├─ deleteNote()
  ├─ selectNote()
  └─ Note class with:
      ├─ encryptContent()
      ├─ decryptContent()
      └─ properties: id, title, content, etc.
```

---

## Component Workflows

### 1. RICHTEXT EDITOR WORKFLOW

**File:** `components/RichTextEditor.jsx`

#### Step 1: User Types in Editor
```javascript
// contentEditable div captures input
<div
  ref={editorRef}
  onInput={handleInput}        // ← Fires on every keystroke
  contentEditable={!isEncrypted}
>
  {content}
</div>
```

#### Step 2: handleInput() Function
```javascript
const handleInput = () => {
  // 1. Get current HTML from editor
  let contentToSave = editorRef.current.innerHTML
  
  // 2. Remove glossary highlights (keep clean)
  contentToSave = contentToSave.replace(
    /<mark class="glossary-highlight-active"[^>]*>(.*?)<\/mark>/gi, 
    "$1"
  )
  
  // 3. Send to parent (Editor.jsx)
  onChange(contentToSave)
  
  // 4. Check grammar
  checkGrammar(editorRef.current.innerText)
}
```

**Why remove marks?** Glossary highlights are visual only. Saved content must be clean.

#### Step 3: Grammar Checking
```javascript
const checkGrammar = (text) => {
  const errors = GrammarChecker.check(text)  // Pattern-based analysis
  setGrammarErrors(errors)                    // Display errors panel
}
```

**GrammarChecker checks for:**
- Repeated words: "the the"
- Spacing issues: "word ,comma"
- Capitalization: sentence starts lowercase
- Common typos

#### Step 4: Save to Parent
```javascript
// In Editor.jsx
const handleContentChange = (newContent) => {
  updateNote(currentNote.id, { content: newContent })  // Save to context
}
```

#### Step 5: Display Grammar Errors
```javascript
{grammarErrors.length > 0 && (
  <div className="border-t border-border bg-surface p-4">
    <h4>Grammar Issues ({grammarErrors.length})</h4>
    {grammarErrors.map((error, idx) => (
      <div key={idx}>
        <span>{error.type}:</span> {error.message}
      </div>
    ))}
  </div>
)}
```

#### Toolbar Functions
```javascript
// Document.execCommand API (legacy but widely supported)
document.execCommand("bold")          // Bold text
document.execCommand("italic")        // Italic
document.execCommand("underline")     // Underline
document.execCommand("foreColor", false, "#FF0000")  // Text color
document.execCommand("fontName", false, "Arial")     // Font
document.execCommand("fontSize", false, "5")         // Size
document.execCommand("justifyCenter")                // Alignment
document.execCommand("insertUnorderedList")          // Bullet list
document.execCommand("removeFormat")                 // Clear formatting
```

---

### 2. GLOSSARY & HIGHLIGHTING WORKFLOW

**File:** `components/RichTextEditor.jsx` + `lib/glossary.js`

#### Step 1: User Clicks 📚 Button
```javascript
<ToolbarButton 
  title="Glossary Terms" 
  onClick={onToggleGlossary}  // ← setShowGlossary(!showGlossary)
>
  📚
</ToolbarButton>
```

#### Step 2: useEffect Detects Change
```javascript
useEffect(() => {
  if (editorRef.current) {
    if (showGlossary) {
      highlightTermsInEditor()  // Turn ON highlights
    } else {
      removeHighlighting()       // Turn OFF highlights
    }
  }
}, [showGlossary])  // ← Dependency array triggers effect
```

#### Step 3: Extract Terms from Content
```javascript
// In lib/glossary.js
export const extractTermsFromText = (text) => {
  const foundTerms = []
  
  // Predefined glossary object
  const glossary = {
    "React": "JavaScript library for UIs",
    "useState": "Hook for managing state",
    "API": "Interface for software communication"
  }
  
  // Check each glossary term against content
  Object.keys(glossary).forEach((term) => {
    const regex = new RegExp(`\\b${term}\\b`, "gi")  // Case-insensitive
    if (regex.test(text)) {
      foundTerms.push(term)
    }
  })
  
  return foundTerms  // e.g., ["React", "useState"]
}
```

#### Step 4: Highlight Terms with `<mark>` Tags
```javascript
const highlightTermsInEditor = () => {
  const terms = extractTermsFromText(content)  // Get found terms
  
  let html = editorRef.current.innerHTML
  
  // Remove old highlights first
  html = html.replace(/<mark class="glossary-highlight-active"[^>]*>(.*?)<\/mark>/gi, "$1")
  
  // Add new highlights
  terms.forEach((term) => {
    // Create safe regex (escape special chars)
    const regex = new RegExp(
      `(?<![<>])\\b${term.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")}\\b(?![^<]*>)`,
      "gi"
    )
    
    // Replace with <mark> wrapper
    html = html.replace(
      regex,
      `<mark class="glossary-highlight-active" data-term="${term}">${term}</mark>`
    )
  })
  
  editorRef.current.innerHTML = html  // Update editor
}
```

**Result:**
```
Before: "React is a library"
After:  "<mark class='glossary-highlight-active'>React</mark> is a library"
CSS:    Yellow background (#fbbf24) on marked terms
```

#### Step 5: Display Glossary Panel
```javascript
{showGlossary && (
  <GlossaryPanel 
    content={content} 
    onClose={() => setShowGlossary(false)} 
  />
)}
```

**GlossaryPanel shows:**
- All found terms in note
- Each term's definition
- Click to expand/view more

#### Step 6: Remove Highlights on Toggle OFF
```javascript
const removeHighlighting = () => {
  const html = editorRef.current.innerHTML
  // Replace <mark>term</mark> with just term (remove markup)
  const cleaned = html.replace(/<mark class="glossary-highlight-active"[^>]*>(.*?)<\/mark>/gi, "$1")
  editorRef.current.innerHTML = cleaned
}
```

---

### 3. AI FEATURES WORKFLOW

**File:** `components/AIPanel.jsx`

#### Step 1: User Clicks "✨ AI Tools" Button
```javascript
// In Editor.jsx
<button 
  onClick={() => setShowAIPanel(true)}
  className="..."
>
  ✨ AI Tools
</button>

// Shows AIPanel modal
{showAIPanel && (
  <AIPanel 
    note={currentNote} 
    onUpdateNote={updateNote}
    onClose={() => setShowAIPanel(false)}
  />
)}
```

#### Step 2: AIPanel Opens with 3 Tabs
```javascript
// Tabs: Summarize | Translate | Generate Tags
<div className="flex border-b border-border">
  <button onClick={() => setActiveTab("summarize")}>Summarize</button>
  <button onClick={() => setActiveTab("translate")}>Translate</button>
  <button onClick={() => setActiveTab("tags")}>Generate Tags</button>
</div>
```

#### Step 3a: SUMMARIZE Feature
```javascript
const handleSummarize = async () => {
  setLoading(true)
  setError("")
  
  // 1. Extract plain text (remove HTML)
  const plainContent = note.content.replace(/<[^>]*>/g, "")
  
  // 2. Validate minimum length
  if (plainContent.trim().length < 50) {
    setError("Content too short to summarize (minimum 50 characters)")
    return
  }
  
  // 3. Send to API
  const response = await fetch("/api/ai/summarize", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ content: plainContent })
  })
  
  // 4. Get response
  const data = await response.json()
  if (data.summary) {
    setSummary(data.summary)  // Display result
  } else {
    setError("Failed to generate summary")
  }
  
  setLoading(false)
}
```

**API Route:** `/app/api/ai/summarize/route.js`
```javascript
// Receives plain text from frontend
// Calls Groq API with prompt
// Returns: { summary: "1-2 sentence summary" }
```

#### Step 3b: TRANSLATE Feature
```javascript
const handleTranslate = async () => {
  setLoading(true)
  
  const plainContent = note.content.replace(/<[^>]*>/g, "")
  
  if (plainContent.trim().length === 0) {
    setError("Add content to translate")
    return
  }
  
  // Send WITH language selection
  const response = await fetch("/api/ai/translate", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      text: plainContent,
      targetLanguage: selectedLanguage  // e.g., "Spanish"
    })
  })
  
  const data = await response.json()
  if (data.translation) {
    setTranslation(data.translation)  // Display result
  } else {
    setError("Failed to translate")
  }
  
  setLoading(false)
}
```

**Supported languages:**
Spanish, French, German, Italian, Portuguese, Russian, Japanese, Chinese

#### Step 3c: GENERATE TAGS Feature
```javascript
const handleGenerateTags = async () => {
  setLoading(true)
  
  const plainContent = note.content.replace(/<[^>]*>/g, "")
  
  // Send BOTH title and content for better context
  const response = await fetch("/api/ai/tags", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      content: plainContent,
      title: note.title  // ← Title helps AI understand context
    })
  })
  
  const data = await response.json()
  if (data.tags && data.tags.length > 0) {
    // IMPORTANT: Tags get SAVED to note
    setTags(data.tags.join(", "))  // Display as string
    onUpdateNote(note.id, { tags: data.tags })  // ← Save to localStorage
  } else {
    setError("Could not generate tags from content")
  }
  
  setLoading(false)
}
```

**Key difference:** Tags are SAVED permanently, Summarize/Translate are display-only.

#### Step 4: API Communication Flow
```
Frontend Button Click
  ↓
Fetch POST to /api/ai/[feature]
  ↓
Backend Route (Next.js)
  ↓
Call Groq API with llama-3.1-8b-instant model
  ↓
Groq processes request
  ↓
Returns AI-generated content
  ↓
Backend returns JSON: { summary/translation/tags: "..." }
  ↓
Frontend receives data
  ↓
Display in modal OR save to note
```

---

### 4. ENCRYPTION WORKFLOW

**File:** `components/EncryptionModal.jsx` + `context/NotesContext.jsx`

#### Step 1: User Clicks "Encrypt Note" Button
```javascript
// In Editor.jsx
{!currentNote.isEncrypted ? (
  <button 
    onClick={() => setShowEncryptionModal(true)}
    className="..."
  >
    🔓 Encrypt Note
  </button>
) : (
  <button 
    onClick={() => setShowEncryptionModal(true)}
    className="..."
  >
    🔒 Manage Encryption
  </button>
)}
```

#### Step 2: EncryptionModal Opens
```javascript
export default function EncryptionModal({ 
  note, 
  onClose, 
  onEncrypt, 
  onRemoveEncryption 
}) {
  const [password, setPassword] = useState("")
  const [confirmPassword, setConfirmPassword] = useState("")
  const [error, setError] = useState("")
  const [showPassword, setShowPassword] = useState(false)
  
  // ... render two password inputs with show/hide toggles
}
```

#### Step 3: User Enters and Validates Password
```javascript
const handleEncrypt = () => {
  // Validation 1: Not empty
  if (!password) {
    setError("Please enter a password")
    return
  }
  
  // Validation 2: Passwords match
  if (password !== confirmPassword) {
    setError("Passwords do not match")
    return
  }
  
  // Validation 3: Minimum length
  if (password.length < 4) {
    setError("Password must be at least 4 characters")
    return
  }
  
  // All validations pass
  onEncrypt(password)  // ← Call parent callback
}
```

#### Step 4: Parent (Editor.jsx) Encrypts Content
```javascript
onEncrypt={(password) => {
  // Call encryptContent method from Note class
  const encryptedContent = currentNote.encryptContent(password)
  
  // Update note with encrypted data
  updateNote(currentNote.id, {
    encryptionPassword: password,  // Store for later decryption
    isEncrypted: true,              // Mark as encrypted
    content: encryptedContent,      // Store encrypted text
    isDecrypted: false,             // Lock it
  })
  
  setShowEncryptionModal(false)  // Close modal
}}
```

#### Step 5: Encryption Method (NotesContext.jsx)
```javascript
encryptContent(password) {
  // CryptoJS AES-256 encryption
  const encrypted = CryptoJS.AES.encrypt(this.content, password).toString()
  return encrypted  // e.g., "U2FsdGVkX1..."
}
```

**How it works:**
- Input: plain text + password
- Output: encrypted gibberish that looks like random characters
- Cannot be decrypted without correct password

#### Step 6: User Opens Encrypted Note
```javascript
// In Editor.jsx - detect encrypted note
if (currentNote.isEncrypted && !currentNote.isDecrypted) {
  setIsPasswordPromptOpen(true)  // Show password modal
}
```

**PasswordPrompt asks:** "Enter password to view this note"

#### Step 7: Decrypt with Password
```javascript
decryptContent(password) {
  if (!this.isEncrypted) {
    throw new Error("Note is not encrypted")
  }

  try {
    // CryptoJS AES-256 decryption
    const decrypted = CryptoJS.AES.decrypt(
      this.content, 
      password
    ).toString(CryptoJS.enc.Utf8)
    
    // Validation: Check if password was correct
    if (decrypted === null || decrypted === undefined || decrypted.length === 0) {
      throw new Error("Incorrect password")
    }
    
    return decrypted  // Plain text
  } catch (error) {
    if (error.message === "Incorrect password") {
      throw error  // Show error message to user
    }
  }
}
```

#### Step 8: Remove Encryption
```javascript
onRemoveEncryption={() => {
  // User clicks "Remove Encryption" button
  // Shows confirmation: "Remove encryption from this note?"
  
  if (confirm("...")) {
    updateNote(currentNote.id, {
      isEncrypted: false,           // Remove encrypted flag
      isDecrypted: false,           // Reset
      encryptionPassword: null,     // Clear password
    })
  }
}}
```

---

### 5. NOTES LIST & CONTEXT WORKFLOW

**File:** `context/NotesContext.jsx` + `components/NotesList.jsx`

#### Step 1: Initialize Context with localStorage
```javascript
// NotesContext.jsx
const [notes, setNotes] = useState([])

useEffect(() => {
  // On mount, load notes from localStorage
  const savedNotes = localStorage.getItem("notes")
  if (savedNotes) {
    const parsed = JSON.parse(savedNotes)
    // Convert plain objects to Note class instances
    const notesWithMethods = parsed.map(note => new Note(note))
    setNotes(notesWithMethods)
  }
}, [])

// Persist whenever notes change
useEffect(() => {
  localStorage.setItem("notes", JSON.stringify(notes))
}, [notes])
```

#### Step 2: Create New Note
```javascript
const createNote = () => {
  const newNote = new Note({
    id: Date.now(),
    title: "Untitled Note",
    content: "",
    tags: [],
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    isEncrypted: false,
    isDecrypted: false,
  })
  
  setNotes([newNote, ...notes])  // Add to front
  return newNote
}
```

#### Step 3: Update Note
```javascript
const updateNote = (id, updates) => {
  setNotes(notes.map(note => {
    if (note.id === id) {
      return {
        ...note,  // Keep existing properties
        ...updates,  // Apply changes
        updatedAt: new Date().toISOString()  // Update timestamp
      }
    }
    return note
  }))
}
```

**Updates can include:**
- `content` - Text change
- `title` - Title change
- `tags` - Tag change (from AI)
- `isEncrypted` - Encryption status
- `encryptionPassword` - Store password

#### Step 4: Delete Note
```javascript
const deleteNote = (id) => {
  setNotes(notes.filter(note => note.id !== id))
}
```

#### Step 5: Select Note
```javascript
const selectNote = (id) => {
  const note = notes.find(note => note.id === id)
  return note
}
```

#### Step 6: NotesList Display
```javascript
// components/NotesList.jsx
export default function NotesList() {
  const { notes, selectNote, deleteNote, createNote } = useNotes()
  const [searchTerm, setSearchTerm] = useState("")
  const [sortBy, setSortBy] = useState("recent")  // recent, oldest, title
  
  // Filter by search
  const filtered = notes.filter(note =>
    note.title.toLowerCase().includes(searchTerm.toLowerCase())
  )
  
  // Sort
  const sorted = filtered.sort((a, b) => {
    if (sortBy === "recent") return new Date(b.updatedAt) - new Date(a.updatedAt)
    if (sortBy === "oldest") return new Date(a.updatedAt) - new Date(b.updatedAt)
    if (sortBy === "title") return a.title.localeCompare(b.title)
  })
  
  return (
    <aside>
      <button onClick={createNote}>+ New Note</button>
      <input 
        placeholder="Search notes..." 
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
      />
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="recent">Most Recent</option>
        <option value="oldest">Oldest First</option>
        <option value="title">By Title</option>
      </select>
      
      {sorted.map(note => (
        <div key={note.id} onClick={() => selectNote(note.id)}>
          <h3>{note.title}</h3>
          <p>{note.content.substring(0, 50)}...</p>
          <small>{new Date(note.updatedAt).toLocaleDateString()}</small>
        </div>
      ))}
    </aside>
  )
}
```

---

## Key Features Explained

### Feature 1: Content Editable
- Browser API for in-place text editing
- No input box needed - user types directly
- Supported by all modern browsers
- Used in Gmail, Google Docs

### Feature 2: Document.execCommand
- Legacy browser API for text formatting
- Commands: bold, italic, underline, etc.
- No external library needed
- Works immediately (no delay)
- Deprecated but still widely supported

### Feature 3: localStorage API
- Client-side data persistence
- Stores up to ~5-10MB per domain
- Survives page refresh
- JSON serialization needed
- No server required

### Feature 4: CryptoJS AES-256
- Client-side encryption library
- AES-256 is military-grade encryption
- Cannot decrypt without password
- Data lost if password forgotten (no recovery)
- Good for privacy, risky for data loss

### Feature 5: Groq API Integration
- LLM (Large Language Model) API
- Model: llama-3.1-8b-instant
- Fast inference (~2-3 seconds)
- Costs money per API call
- Used for summarize, translate, tags

### Feature 6: Regex Highlighting
- Pattern matching for glossary terms
- Case-insensitive search
- Whole-word matching (don't match "state" in "statement")
- Negative lookahead/lookbehind prevents matching inside tags
- Wraps matches with `<mark>` HTML tag

---

## Interview Q&A

### Question 1: Tell me about your project architecture

**Answer:**
"NoteFlow AI is a Next.js-based note-taking app with three main layers:

1. **Presentation Layer** - React components (Header, Editor, NotesList, modals)
2. **State Layer** - React Context API for global state management, stores notes in an array
3. **Persistence Layer** - localStorage for client-side data storage, survives page refresh

Data flow is: User Input → Component → Context → localStorage → Display

The app has one main context (NotesContext) that manages all notes. Each note is a Note class instance with methods like encryptContent() and decryptContent(). Components subscribe to this context and re-render when notes change."

---

### Question 2: How do you handle rich text editing?

**Answer:**
"I use React's contentEditable attribute on a div element. Here's the flow:

1. Div with `contentEditable={!isEncrypted}` lets users type directly
2. Every keystroke fires `onInput` event
3. I extract the innerHTML (which includes formatting)
4. Before saving, I remove temporary highlights (glossary marks)
5. Send clean HTML to parent component
6. Parent saves to Context, which persists to localStorage

Text formatting comes from document.execCommand API - calls like execCommand('bold'), execCommand('italic'), etc. This is legacy but widely supported and immediate (no delay).

The key insight: We save and display the full HTML (including `<b>`, `<mark>` tags), not plain text. This preserves formatting."

---

### Question 3: How does encryption work?

**Answer:**
"I use CryptoJS library for AES-256 encryption. Here's the flow:

**Encrypting:**
1. User enters password in modal (validated: not empty, min 4 chars, matches confirmation)
2. Call `note.encryptContent(password)`
3. This uses CryptoJS.AES.encrypt(plainText, password) and returns encrypted gibberish
4. Store encrypted string in note.content with `isEncrypted: true` flag
5. Save to localStorage

**Decrypting:**
1. When user opens encrypted note, detect `isEncrypted: true`
2. Show password prompt modal
3. User enters password
4. Call `note.decryptContent(password)`
5. This uses CryptoJS.AES.decrypt(encryptedText, password)
6. If password correct → returns plain text
7. If password wrong → returns empty/null → show error
8. Set `isDecrypted: true` so note becomes editable

The key limitation: No password recovery. If user forgets password, data is permanently lost because encryption is on the client side only."

---

### Question 4: Explain the glossary highlighting feature

**Answer:**
"Glossary lets users highlight vocabulary terms in their notes. Flow:

1. **Predefined glossary** - We have a static glossary object with terms as keys, definitions as values
2. **User clicks 📚 button** - Sets `showGlossary: true`
3. **Extract terms** - `extractTermsFromText(content)` loops through glossary terms and checks which ones appear in the note
4. **Highlight terms** - For each found term, create a regex pattern and wrap it: `<mark class='glossary-highlight-active'>term</mark>`
5. **CSS provides yellow background** - The mark class has `background-color: #fbbf24`
6. **Show panel** - GlossaryPanel displays all found terms with definitions

**Important detail**: When user types, we remove these highlights before saving (replace `<mark>term</mark>` with just `term`) because highlights are visual only. Next time user opens the note and clicks 📚, highlights regenerate.

Why? Saving clean content keeps the data pristine and highlights can be reapplied dynamically."

---

### Question 5: How does the AI integration work?

**Answer:**
"I use Groq API (llama-3.1-8b-instant model) for AI features. Three main features:

**1. Summarize:**
- Strip HTML from content
- Validate minimum 50 characters
- Send plain text to `/api/ai/summarize`
- Groq returns 1-2 sentence summary
- Display in modal (not saved)

**2. Translate:**
- Strip HTML from content
- User selects target language from dropdown
- Send text + language to `/api/ai/translate`
- Groq translates to that language
- Display in modal (not saved)

**3. Generate Tags:**
- Strip HTML from content
- Send BOTH title and content to `/api/ai/tags`
- Groq analyzes and returns relevant tags array
- Display as badges
- **SAVE to note** - This is the only AI feature that modifies the note permanently

Flow: Frontend sends JSON → Backend route receives → Calls Groq API → Returns response → Frontend displays

Why different routes? Each feature needs different prompts and processing logic."

---

### Question 6: How do you manage state and persistence?

**Answer:**
"State management uses React Context API:

1. **Create Note class** - With properties (id, title, content, tags, isEncrypted, etc.)
2. **Create NotesContext** - Global state for notes array
3. **useNotes hook** - Provides access to notes and functions (createNote, updateNote, deleteNote, selectNote)
4. **Two useEffects:**
   - On mount: Load from localStorage
   - On notes change: Save to localStorage (auto-persist)

**Update flow:**
```javascript
updateNote(id, updates) {
  setNotes(notes.map(note =>
    note.id === id 
      ? { ...note, ...updates, updatedAt: now }
      : note
  ))
}
```

This is a pure function approach - create new array instead of mutating. React detects change and re-renders. localStorage useEffect then persists.

Key insight: Immutable updates help React detect changes and enable time-travel debugging if needed."

---

### Question 7: How do you handle grammar checking?

**Answer:**
"Grammar checking is pattern-based (not AI):

1. Every keystroke in editor fires `onInput`
2. Extract plain text from innerHTML
3. Call `GrammarChecker.check(text)` which checks for patterns:
   - Repeated words: /\\b(\\w+)\\s+\\1\\b/ matches 'the the'
   - Spacing issues: /\\s+[,\\.!?]/ matches 'word , comma'
   - Capitalization: check if sentence starts with lowercase
   - Common typos: hardcoded list

4. Return array of errors: [{type: 'Repeated Word', message: '...'}]
5. Display in red panel at bottom of editor

**Note:** This is NOT machine learning - just regex patterns. It's fast (instant) but limited. For production, you'd use a real grammar API like Grammarly.

User can't auto-fix - they must manually edit. Grammar panel just alerts them."

---

### Question 8: What's your approach to responsive design?

**Answer:**
"I use Tailwind CSS responsive prefixes:

- `hidden md:flex` - Hide on mobile, show on medium screens (768px+)
- `w-full md:w-64` - Full width mobile, 64 units on desktop
- `p-4 md:p-6` - Smaller padding mobile, larger on desktop
- `gap-0 md:gap-4` - No gap mobile, gap on desktop

Layout:
- **Mobile**: Single column, sidebar hidden
- **Desktop**: Two column - sidebar + main area

Key components:
- Header - Top navigation, always visible
- Sidebar (NotesList) - Hidden on mobile, flex on md:
- Main editor - Takes full width or flex-1
- Modals - Fixed, centered, work on all screens

Tested with mobile, tablet, desktop breakpoints."

---

### Question 9: How would you improve this project?

**Answer:**
"Several areas for improvement:

1. **Dark mode implementation** - CSS variables are defined, ThemeProvider is ready. Need to:
   - Wrap app with ThemeProvider in layout
   - Add theme toggle button in Header
   - Use useTheme hook to switch

2. **User authentication** - Currently no login, anyone can access notes. Need:
   - User registration/login
   - Server-side validation
   - JWT tokens

3. **Cloud sync** - Currently localStorage only. Could add:
   - Backend database (Firebase, Supabase)
   - Sync across devices
   - Backup and recovery

4. **Custom glossary** - Currently static, could add:
   - Form to add/edit glossary terms
   - Save custom glossary to localStorage
   - Share glossary with others

5. **Better grammar checking** - Use real API:
   - Grammarly API
   - LanguageTool API
   - ML-based detection

6. **Performance** - Could optimize:
   - Memoization (React.memo, useMemo)
   - Code splitting
   - Lazy loading components
   - Image optimization

7. **Testing** - Add:
   - Unit tests (Jest)
   - E2E tests (Cypress)
   - Coverage reports"

---

### Question 10: Explain the data flow for a note edit

**Answer:**
"Step by step:

1. **User types in editor**
   - onInput fires
   - handleInput() called

2. **Extract and clean content**
   - Get innerHTML from contentEditable div
   - Remove glossary `<mark>` tags
   - Result: clean HTML

3. **Send to parent**
   - onChange(cleanHTML) called
   - Handler: handleContentChange(newContent)

4. **Update context**
   - updateNote(id, { content: newContent })
   - Context recalculates notes array with spread: `{ ...note, content: newContent, updatedAt: now }`

5. **Re-render**
   - Context subscribers re-render
   - All components using notes get new data

6. **Persist to localStorage**
   - useEffect in Context detects notes change
   - localStorage.setItem('notes', JSON.stringify(notes))
   - Runs every time notes array changes

7. **Auto-save complete**
   - No 'Save' button needed
   - User sees data persisted on refresh

Flow: User → Component → Context → localStorage → Persisted"

---

## Quick Reference

### Common Interview Answers (Short)

**Q: What tech stack?**
A: React 19 + Next.js 16, Tailwind CSS, Context API for state, localStorage for persistence, CryptoJS for encryption, Groq API for AI.

**Q: Why localStorage?**
A: Simple client-side persistence, no backend needed for MVP, survives page refresh.

**Q: Why CryptoJS?**
A: Works in browsers, AES-256 is strong encryption, easy to integrate.

**Q: Why Context API?**
A: Avoids prop drilling, simpler than Redux for this size project.

**Q: Why contentEditable?**
A: Allows in-place text editing without input element, used by Gmail and Google Docs.

**Q: How do you save data?**
A: Every change updates Context, which triggers useEffect that saves to localStorage.

**Q: What's the biggest challenge?**
A: Handling encrypted content + glossary highlights together - had to ensure highlights don't get saved in encrypted state.

---

### Function Reference

| Function | File | Purpose |
|----------|------|---------|
| `handleInput()` | RichTextEditor.jsx | Save editor content on keystroke |
| `highlightTermsInEditor()` | RichTextEditor.jsx | Wrap glossary terms with `<mark>` |
| `removeHighlighting()` | RichTextEditor.jsx | Remove `<mark>` tags |
| `extractTermsFromText()` | glossary.js | Find glossary terms in content |
| `GrammarChecker.check()` | GrammarChecker.jsx | Detect grammar errors |
| `handleEncrypt()` | EncryptionModal.jsx | Validate and encrypt |
| `encryptContent()` | NotesContext.jsx | CryptoJS AES encrypt |
| `decryptContent()` | NotesContext.jsx | CryptoJS AES decrypt |
| `updateNote()` | NotesContext.jsx | Update note in context |
| `createNote()` | NotesContext.jsx | Create new note |

---

### Key CSS Classes

| Class | Purpose | Color |
|-------|---------|-------|
| `glossary-highlight-active` | Glossary term highlight | Yellow (#fbbf24) |
| `bg-destructive` | Error/delete color | Red |
| `bg-primary` | Main action color | Blue |
| `bg-surface` | Secondary background | Dark gray |
| `text-foreground` | Main text color | White |

---

### Keyboard Shortcuts (document.execCommand)

```
Ctrl+B / Cmd+B  → Bold
Ctrl+I / Cmd+I  → Italic
Ctrl+U / Cmd+U  → Underline
```

---

### Regex Patterns Used

```javascript
// Remove HTML tags
/<[^>]*>/g

// Glossary highlights
/<mark class="glossary-highlight-active"[^>]*>(.*?)<\/mark>/gi

// Repeated words (grammar)
/\b(\w+)\s+\1\b/gi

// Spacing before punctuation (grammar)
/\s+[,\.!?]/g

// Whole word match (glossary extraction)
/\bterm\b/gi
```

---

## Interview Tips

1. **Start with high-level overview** - Explain architecture first, then dive into specifics
2. **Use visual descriptions** - "Flow is: User input → Component → Context → localStorage"
3. **Give code examples** - Show small snippets while explaining
4. **Mention trade-offs** - "We chose Context API over Redux for simplicity"
5. **Be honest about limitations** - "Password encryption means if user forgets password, data is lost"
6. **Ask clarifying questions** - "Do you want to know more about how X works?"
7. **Practice saying it aloud** - Record yourself explaining each feature
8. **Have follow-up answers ready** - Expect "Why not use X instead of Y?"

---

## What To Say When They Ask About X

### "Why not use Redux?"
"For this size project, Context API is sufficient. Redux is better for very large apps with complex state. Context is simpler and has less boilerplate."

### "Why not use a database?"
"Wanted to build a working MVP first. localStorage is quick to implement. For production, would add Firebase or Supabase for sync across devices."

### "Why not use an existing rich text editor?"
"contentEditable is native browser API with full control. Libraries like Slate.js add overhead. For this case, simpler to build from scratch."

### "How do you handle security?"
"Encryption is client-side, so no password stored on server. For production, would add server-side validation and authentication. Token-based auth with JWT."

### "What if user loses password?"
"Data is permanently lost - that's a limitation of client-side encryption. For real apps, would offer password recovery via email or backup codes."

---

**Last Updated:** December 1, 2025
**Status:** Ready for Interview
**Confidence Level:** High
