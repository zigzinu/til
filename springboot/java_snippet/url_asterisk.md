# How does asterisk(*) work in java URLs?

/welcome* : anything in THIS folder or URL section, that starts with "/welcome" and ends before next "/" like /welcomePage.

/welcome** : any URL, that starts with "/welcome" including sub-folders and sub-sections of URL pattern like /welcome/section2/section3/ or /welcomePage/index.

/welcome/* : any file, folder or section inside welcome (before next "/") like /welcome/index.

/welcome/** : any files, folders, sections, sub-folders or sub-sections inside welcome.

In other words one asterisk * ends before next "/", two asterisks ** have no limits.

Reference
- https://stackoverflow.com/questions/40188743/double-asterisk-in-a-request-mapping
