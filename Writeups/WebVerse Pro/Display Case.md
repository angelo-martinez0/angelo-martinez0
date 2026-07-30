# Display Case
> Display Case is the file-sharing tool a four-person agency uses to swap rough cuts and project archives with their clients. They moved off Dropbox to "control the surface". The site lists exactly the files the team intended to publish, the folder underneath them does not. Start at the landing page, look around the uploads area, and see what a stray legacy panel will hand you if you ask.

---

### Objective
#### Get information on the files that have been uploaded
---
## Reconnaisance
#### In the dashboard we could easily see in the description mention about the files into `/uploads/`

![alt text](/Writeups/images/images.png)


#### Using Sitemap we can easily see the directories inside the domain we are investigating in that way we can find easily what we are looking for especially hidden directories we can go back anytime 

![alt text](/Writeups/images/images-1.png)

---

In Upload it says files land at `/uploads/`; so what will happen if we traverse to `/uploads/`
![alt text](/Writeups/images/images-2.png)


#### Now we can see the files that has been uploaded.

![alt text](/Writeups/images/images-3.png)

> A directory `_legacy_admin/` can be seen. Going inside it will soon reveal where the `flag` has been stored.

![alt text](/Writeups/images/images-4.png)