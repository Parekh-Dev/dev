# 🚀 QUICK REFERENCE - 5 MINUTE READ BEFORE INTERVIEW

## ONE-LINE PROJECT DESCRIPTION
"NoteFlow-AI is an AI-powered note-taking app with encryption, real-time editing, and smart features like summarization and translation."

---

## TECH STACK (What to say)
```
Frontend: Next.js + React + Tailwind CSS + Radix UI components
Backend: Node.js (Next.js API routes)
State: React Context API + localStorage
Security: CryptoJS (AES encryption)
AI: Groq API (llama-3.1)
Hosting: Vercel
```

---

## FEATURES (Things you can demo)
✅ Create/Edit/Delete notes
✅ Rich text editing with formatting
✅ Encryption with password protection
✅ AI Summarization (condense notes to 1-2 sentences)
✅ AI Translation (8 languages)
✅ AI Tag Generation (auto-create relevant tags)
✅ Dark/Light theme toggle
✅ Pin important notes
✅ Responsive mobile design

---

## FILE STRUCTURE TO KNOW
```
/app/api/ai/*           ← API routes (summarize, translate, tags)
/components/Editor.jsx  ← Main editing interface
/components/AIPanel.jsx ← AI tools modal
/context/NotesContext   ← Global state (all notes + current note)
/components/ui/*        ← Reusable UI components
```

---

## DATA FLOW (Remember this order)
```
User edits note
    ↓
handleContentChange() triggered
    ↓
updateNote() called
    ↓
Note saved to localStorage
    ↓
React state updated
    ↓
UI re-renders
```

---

## ENCRYPTION FLOW
```
Encrypt button clicked
    ↓
User enters password
    ↓
CryptoJS.AES.encrypt(content, password)
    ↓
Encrypted string stored in note.content
    ↓
isEncrypted = true, isDecrypted = false
    ↓
Next load → PasswordPrompt modal appears
```

---

## AI FLOW
```
User clicks "Translate to Spanish"
    ↓
Frontend sends POST to /api/ai/translate
    ↓
{text, targetLanguage} sent to Groq API
    ↓
Groq API calls llama-3.1 model
    ↓
Returns translation
    ↓
Display in modal
```

---

## IMPORTANT CODE LOCATIONS
```
Note creation:    NotesContext.jsx → createNote()
Note updates:     NotesContext.jsx → updateNote()
Encryption:       NotesContext.jsx → Note.encryptContent()
Decryption:       NotesContext.jsx → Note.decryptContent()
Editor UI:        Editor.jsx
AI Tools UI:      AIPanel.jsx
Translate API:    /api/ai/translate/route.js
Summarize API:    /api/ai/summarize/route.js
Tags API:         /api/ai/tags/route.js
```

---

## TOP 5 QUESTIONS THEY MIGHT ASK

**Q1: Why Context API and not Redux?**
A: "Project is small. Context API is simpler. If app grew to 10+ pieces of global state, I'd consider Zustand or Redux."

**Q2: How does encryption work?**
A: "Client-side AES encryption with CryptoJS. Password never leaves browser. If they forget password = data lost."

**Q3: What if GROQ_API_KEY is missing?**
A: "The translation fails and returns error. I fixed it to return proper error instead of fake translation."

**Q4: How is data persisted?**
A: "localStorage. Every edit triggers saveNotes(). Cleared if user clears browser data. Next step = real database."

**Q5: What would you improve?**
A: "1. Database + auth (Supabase) 2. Real-time sync with WebSockets 3. Version history 4. Offline support with Service Workers 5. Better rich editor library"

---

## COMMON MISTAKES TO AVOID
❌ Don't say "I don't know" - say "I haven't implemented that yet, but I would..."
❌ Don't make up technical details - stick to what you actually built
❌ Don't spend >1 min on one question - ask for hint if stuck
❌ Don't forget to mention error handling and edge cases
❌ Don't skip the "why" - explain your design choices

---

## DEMO FLOW (7 minutes)
1. Show home screen + sidebar (10 sec)
2. Create note + edit (20 sec)
3. AI Summarize (20 sec)
4. AI Translate (20 sec)
5. Encrypt note (30 sec)
6. Try wrong password (10 sec)
7. Unlock with correct password (10 sec)

**Total: 2 minutes demo** ← Leave time for questions

---

## IF THEY ASK YOU TO CODE

**Small task #1: "Add a note background color"**
```
1. Add color: 'white' to Note class
2. Add <select> in Editor for color
3. Call updateNote(id, {color: newColor})
4. Add className based on color prop
5. Save to localStorage (automatic)
```

**Small task #2: "Add search in sidebar"**
```
1. Add searchTerm state in NotesContext
2. Filter notes: notes.filter(n => n.title.includes(searchTerm))
3. Add input field in NotesList component
4. Display filtered results
5. Handle empty results
```

**Small task #3: "Prevent data loss on page close"**
```
1. Add window.beforeunload listener
2. Show warning: "You have unsaved changes"
3. Call saveNotes() before unload
4. Let browser handle save
```

---

## CONFIDENCE BOOSTERS
✅ You've built the entire app yourself
✅ You understand every component
✅ You've diagnosed and fixed bugs
✅ You're aware of limitations and improvements
✅ You can explain technical decisions
✅ You write clean, understandable code

**You've got this!** 💪

---

## VERCEL DEPLOYMENT CHECKLIST
- [ ] Environment variable GROQ_API_KEY added to Vercel
- [ ] Project is deployed and working
- [ ] Translation works on live site
- [ ] All features functional on Vercel
- [ ] No console errors in browser DevTools

If interviewer visits Vercel link, everything should work smoothly!
