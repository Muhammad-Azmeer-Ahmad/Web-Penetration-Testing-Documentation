# Lab Notes — Module 01 Web Requests

## What I Actually Did
- Sent first curl request to info.cern.ch, saw raw HTML in terminal
- Tested Basic Auth three ways — all worked but manual header felt
  most useful to understand what the browser sends automatically
- Forgot -L on POST login, kept getting 302 redirect instead of page
  Added -L and it followed through
- Tried JSON POST without Content-Type header — server rejected it
  even with valid cookie. That one stuck with me.
- Ran all four CRUD operations on the API — deleted the city I created
  and confirmed it was gone with a GET after

## What Clicked
Cookie alone = authenticated. No password needed after login.
That is session hijacking in one sentence.

## What Confused Me
Why PUT needs the entity in the URL but POST does not.
Makes sense now — POST creates new, PUT targets existing row.

## What I Would Check First On a Real Target
Every input field and URL parameter for GET.
Login form for POST — capture the cookie immediately after.