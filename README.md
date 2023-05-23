# Team Aurous Website

This repository contains the files and source code for the Team Aurous website and EVFS wiki.

## Wiki/documentation

The Wiki should be organized as so:
- Inside the source folder, each "channel" or "topic" should be its own folder
  - Its index should be a table of contents for each subtopic
  - For each subtopic, it should be its own markdown file
  - If a subtopic is a wiki, it should try its best to explain everything and include references

The Wiki is build with [`Retype`](https://retype.com/). To install:
1. Make sure the [`dotnet SDK`](https://dotnet.microsoft.com/en-us/download), [`node.js`](https://nodejs.org/en), or [`yarn`](https://classic.yarnpkg.com/en/) installed.
2. Install `Retype`:
   - For `node.js`, `npm install retypeapp --global`
   - For `Yarn`, `yarn global add retypeapp`
   - For `dotnet`, `dotnet tool install retypeapp --global`
3. Start updating the docs:
   - For having a live-reload server: `retype watch`
   - To build a final static website: `retype build`
