- **Purpose:** To store your raw notes directly from the sources (YouTube videos, Podcasts, Books, Readwise highlights). These are your "Literature Notes."
    
- **Structure:**
    
    - `40_Sources/Books/`
        
    - `40_Sources/Courses/`
        
    - `40_Sources/YouTube_Podcasts/`
        
    - `40_Sources/Articles_Papers/`
        
- **Content (for each source):**
    
    - A single note per source (e.g., `Book_Clean_Code_Robert_Martin.md`, `Podcast_Lex_Fridman_AI_Safety.md`, `YT_Channel_FreeCodeCamp_Python_Tutorial.md`).
        
    - **Metadata (Frontmatter):** Use YAML frontmatter for properties like `type: book`, `author: Robert C. Martin`, `source: YouTube`, `date_consumed: 2025-07-29`, `status: processed/in_progress`.
        
    - **Highlights/Summaries:** Paste your Readwise highlights here. Add your own brief summaries of key sections from videos/podcasts.
        
    - **Links to Concepts:** Crucially, from these source notes, you will _link out_ to your atomic notes in `30_Concepts`. This is how you process raw information into your own knowledge base.
        
        - Example: In your "Clean Code" source note, you might have a highlight about "meaningful names." You'd then link `[[Meaningful Names]]` which is an atomic note in your `30_Concepts` folder.