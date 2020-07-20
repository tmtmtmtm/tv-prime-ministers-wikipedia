Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the [Prime Minister of Tuvalu](https://www.wikidata.org/wiki/Q592866)
looks fine structurally, although it was out of date as to who the
current officeholder is.

Step 2: Tracking page
=====================

PositionHolderHistory page created at https://www.wikidata.org/w/index.php?title=Talk:Q592866&oldid=1111067752

Current status 6 dated officeholders, and 2 dated; 12 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit the [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = [Prime Minister of Tuvalu](https://en.wikipedia.org/wiki/Prime_Minister_of_Tuvalu)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

This is a fairly simple table so only needed a few small tweaks
(though needed to be careful to only take the *second* table) as
the first is for the Chief Minister of the Ellice Islands. As that only
had one officeholder, I might add it manually at the end.

* Later update: Hah! That's not so simple. See thread at
https://twitter.com/tmtm/status/1285141272777433088

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

9 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/304811da27cff/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

5 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/78eac187768f6/

I also accepted the suggested change of date for Saufatu Sopoanga:

    wd uq '384322$28D9A6D6-B1B0-41B2-8EB1-6ABBEB8E0F38' P580 2002-08-02 2002-08-24

Step 8: Refresh the Tracking Page
=================================

Final version: https://www.wikidata.org/w/index.php?title=Talk:Q592866&oldid=1235139237


