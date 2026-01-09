## SET|Files

### One-Page Protocol

Parsing - SET files are initially parsed on NewLine (LF or CRLF)
    
Preamble (optional) - comments containing information about the file and its content  
    `myconfig.set` (filename of the document - strongly recommended as the first line)  
    `The "one-page" definition of SET files`
    
*   **Group(s)** – Sections of the file containing related information, beginning with the Group name
    `[GROUPS]`
    
    *   Each Group MUST have a unique name within the file - enclosed in square brackets []

    *   May only contain Letters, Numbers, hyphens `-`, and underscores `_`

    *   A Group may contain diverse types of information: Key|Value, Table, Delimited String
        
    *   A Text Group may be used to store extensive text information (even multi-line)

    *   Groups end with an empty line, the next group, `[EOG]` End-of-Group, or `[EOF]` End-of-File
        
*   **\[THIS-FILE**: a special Group to contain internal settings for the file / file-parser  
        If used, this should be the first Group in the file
        `[THIS-FILE]`  
        `Version|4.2`  
        `Created|2025-11-27`
        `Delimiter|:[]:{}:|:\:…:!`
    
*   **Text-Group(s)**: a special type of Group  
    `[{TEXT-GROUPS}]`
    
    *   All contents of a Text-Group are a single entity allowing multiple lines and unusual characters
        
    *   No escaping is needed inside a Text-Group, all content is "as-is"
        
    *   Unlike other Groups Text-Groups do not end on empty lines.
        
    *   Text Group names may be use in a Line, as a Linked Reference - 
        `License | [{LICENSE}]`
        
*   **Line(s)**: within a group hold information in delimited arrays.  
    `Protocol|RS232|9600|8|N|1|none`
    
    *   A Line could be a single entry array. 
        The parser might look for "line 3" of `[CONFIG]` – reading the whole line, no delimiter
        
    *   A Line may be a `Key|Value` pair - one delimiter
        
    *   A Line may be a delimited array of data like a csv - but normally `|` pipe delimited
        
    *   A field in a line, may reference a Text Group in the same SET File
        
    *   Spaces before/after delimiters may provide readability but the parser must account for its use, or not, as needed
        
    *   A field within a line may itself be an array of data (nested array), 
        by simply using the secondary delimiter (default=`!`)
        
*   **Comments**: Any text outside of a Group is considered a comment
    
    *   Any comment immediately preceding a Group is assumed to be related to that group


SET_File_One-Page_v4.2 – www.setfiles.org    