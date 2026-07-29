# Corridor

## Foundational


#### Ridgeline Press is a three-person independent literary press in the Pacific Northwest. They publish a quarterly journal of fiction, essays, and poetry, and their site is a static-as-it-gets PHP app that loads each piece from a file on disk. No CMS, no database, just a folder of HTML fragments and one unlucky `readfile` call. Read the site. Find what shouldn't be there.

## Enumeration
- Reviewing the categories on the homepage reveals that navigating into any of the `Current Issue` pages exposes a URL structure that has `.php?slug=?`

 ![alt text](/Writeups/images/image1.png)

 - Which means it can direct us to the `directory` of the html

 - First thing to test is going back one directory using `../`

 ![alt text](/Writeups/images/image2.png)
