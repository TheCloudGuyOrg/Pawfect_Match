# Pikes Pack (dog playdate app): Spec and Working Notes

Status: **scoping only. Nothing built, nothing deployed.** Captured 2026-10-29 from a planning conversation. Website first; a phone app comes later.

## 1. Decisions made

| Question | Answer |
|---|---|
| Concept | Dating-app style matching **for dogs**: dogs match with other dogs of similar breed and temperament, then their owners arrange a meetup |
| Animals | **Dogs only** |
| Launch area | **Colorado Springs, CO** |
| Nature of project | **Hobby project** |
| Revenue | None for now; just a concept |
| Name / branding | Working name **Pikes Pack** (placeholder; likely to change). Availability not checked |
| Platform | Website first, phone app later |
| Intact dogs | Each dog profile has an **intact / spayed-neutered flag**, and owners can **filter and sort by it** in Discover |
| Scope | **Playdates only for now.** Breeding matches are out of scope and may be revisited later (see section 9) |
| Meetup locations | **Public dog parks only** |
| Vaccination | Status shown on the profile (owner-declared, not verified) |
| Hosting | **AWS** |
| Park data | Small hand-curated Colorado Springs list; see section 7 |
| Branding / revenue | Neither yet; just a concept |

## 2. Core concept

Each owner creates a profile for their dog. The app shows other dogs nearby, one at a time, in a swipe-style stack. Owners like or pass. When **both** owners like each other's dog, it's a match. A match opens a chat and suggests a meetup at a public dog-friendly place in Colorado Springs.

The humans are the ones meeting, so the app should be clear that it is for dogs to make friends, not for owners to date, and should be built with owner safety in mind (section 8).

## 3. Minimum first version

1. **Accounts:** sign up, sign in, email verification, 18+ confirmation.
2. **Dog profile:** photos, name, breed (or mix), size, age, sex, spayed/neutered, vaccinations up to date (owner-declared), and a short bio.
3. **Temperament questionnaire** (structured, not free text; see section 4).
4. **Discover:** swipe-style stack of nearby dogs, with filters for distance, size and age.
5. **Matching:** a match is created when both owners like each other's dog. Passes are hidden.
6. **Chat:** open to matched owners only.
7. **Meetup suggestion:** pick a public place and a time from within the chat, and the other owner accepts or counters.
8. **Safety tools:** report, block, community rules, and an admin view for reports.

## 4. Matching (suggestions to confirm)

"Similar breed and temperament" is the goal. A suggestion is to treat **temperament and size as the main signals** and breed as a softer one, because compatibility is mostly about play style, energy and size rather than breed. Breed groups (for example herding, terrier, toy) can still add to the score.

**Temperament questions (approved for now; may be refined after testing with real users).** All are owner-reported and unverified. Aim for under three minutes in onboarding. Consider having a local Colorado Springs trainer review them.

| # | Question | Answers | Use |
|---|---|---|---|
| 1 | Energy level | Low / Medium / High | Soft score (within one step) |
| 2 | Play style (pick any) | Chase / Wrestle / Fetch or ball / Tug / Gentle sniff-and-stroll / Not into playing | Soft score (overlap) |
| 3 | Play intensity | Gentle / Moderate / Rough | Soft score; filter extremes |
| 4 | Comfortable playing with (pick any) | Small (under about 25 lb) / Medium / Large | **Hard filter** on both sides |
| 5 | Meeting new dogs | Eager greeter / Polite / Shy, warms up slowly / Selective | Soft score; shown on card |
| 6 | Any history of snapping, biting or fights with dogs? | None / Minor (describe) / Yes (describe) | **Hard filter**; reviewed on report |
| 7 | Reactive on leash? | No / Sometimes / Yes | Hard filter for owners who want relaxed dogs |
| 8 | Guards toys, treats or people? | No / Sometimes / Yes | Flag on card; tips before meetup |
| 9 | Group size | One-on-one / Small group / Busy park | Soft score |
| 10 | Off-leash recall | Reliable / Sometimes / Leash only | Shown on card; used to suggest park type |
| 11 | Life stage | Puppy (under 1) / Adult / Senior | Soft score; puppy rules below |
| 12 | Limits | Joint or mobility issues, afraid of loud noises, etc. | Shown on card |

Also on the profile: spayed/neutered flag, vaccination status, and for intact females an **"in heat" toggle** that blocks meetup scheduling until cleared. Puppies should not be offered park meetups until their vaccination series is complete (general veterinary guidance; confirm wording with a vet).

**Matching summary:** hard filters first (distance, size comfort both ways, bite history vs "relaxed dogs only", reactivity, intact setting, in heat), then a soft score on energy, play style, intensity, group size and age band, with breed group a minor factor. Show the reasons on the card ("both high energy, both like fetch").

**After each meetup,** ask each owner: "Got along / Not a fit / Safety concern". This improves matching and gives a safety signal for moderation.

How it would rank:
1. **Hard filters first:** distance, size compatibility range, and anything the owner has ruled out. Do not show a reactive dog to someone who wants only relaxed dogs.
2. **Then a weighted score:** play style, energy, size, age band, then breed group.
3. Keep it simple and explainable at first ("both high energy, both like fetch"). More advanced ranking can wait until there is real data.

**Intact dogs (decided):** the intact flag is a normal profile field. Owners choose in their preferences whether to see only spayed/neutered dogs, only intact dogs, or both, and can sort Discover by it. Default for new users: **spayed/neutered only** (decided). Still to decide: whether intact dogs get an extra reminder before a meetup.

## 5. Pages

Landing, sign up / sign in, onboarding (profile and questionnaire), discover (swipe), matches list, chat, meetup planner, my dog, edit profile, owner settings, safety/community rules, admin moderation.

## 6. Data model

- **User:** id, email, display name, approximate location, 18+ confirmed, status
- **Dog:** owner, name, breed, size, age, sex, neutered, vaccinated, temperament answers, bio, photos
- **Swipe:** from dog, to dog, like/pass, time
- **Match:** two dogs, created when both like, status
- **Message:** match, sender, text, time
- **Place:** name, type (dog park, trail, etc.), coordinates, notes
- **Meetup:** match, place, time, status (proposed / accepted / done / cancelled)
- **Report / Block:** reporter, target, reason, status

## 7. Technical approach

- **Backend as an API first,** so the phone app can reuse it later. The website is one client of that API.
- **Responsive website** that works well on a phone browser; installable as a web app later if wanted.
- **AWS option** (the owner already uses AWS): Cognito for accounts, API Gateway + Lambda for the API, DynamoDB for data, S3 + CloudFront for photos, SES for email. Maps and address search through Amazon Location Service or Google Maps (the most variable cost).
- **Alternative:** a hosted backend such as Supabase (built-in sign-in and location search) gets to a first version faster, but lives outside AWS. Decide based on how much the owner wants to run and learn on AWS.
- **Frontend:** a modern JavaScript framework such as React or Next.js; React Native could share code with the later phone app. Not decided.
- **Hosting (decided): AWS.** Build the API first, then the web page.
- **Park data (recommended approach):** keep a **small hand-curated list** of Colorado Springs dog parks, roughly 15 to 30, stored in the app's own database. Hobby scale does not justify a paid places service.
  1. **Primary source:** the City of Colorado Springs Parks, Recreation & Cultural Services dog-parks page (https://parks.coloradosprings.gov/parks/page/dog-parks). It says City Code requires leashes in city parks and on trails except in designated off-leash areas, so only designated off-leash parks should be offered for meetups. Some parks are not run by the city (for example Falcon Dog Park), so check the county and other operators as well.
  2. **Coordinates and cross-check:** OpenStreetMap, which tags dog parks as `leisure=dog_park` and can be queried with the free Overpass API. OSM data is under the ODbL license, which requires attribution and has conditions on sharing a derived database. **Read the license terms before storing OSM data in the app.** Alternatively, look up each park's coordinates once by hand.
  3. **Verify by hand:** check name, address, hours, fencing, small-dog area and rules from the official page or by visiting. Websites such as local tourism pages and blogs are useful leads but not authoritative.
  4. **Avoid at first:** copying data from Google Places or similar services into your own database. I recall their terms limit storing and caching their data, and Amazon Location Service has separate rules for storing results. Check the terms before relying on either. Using a map service only to **display a map or find an address** is fine.
  5. **Growth:** later let users suggest a park, with admin approval.
  Example parks to check (from search results, not verified): Cheyenne Meadows, Palmer Park, Rampart, Antlers, Falcon Dog Park, and Portal Park (which had an off-leash survey).
- **Launch area:** limit signups and matching to Colorado Springs / El Paso County at first.

## 8. Safety, privacy and legal (important even for a hobby)

- **Owner safety:** public places only, daytime suggestions, first-meeting tips, no exact addresses, report and block everywhere.
- **Privacy:** show approximate distance only, never a home address. Keep the minimum personal data and have a short privacy notice.
- **Photos:** upload size limits and a moderation plan (reports first; automated image checks such as Amazon Rekognition could be added).
- **Dog safety:** vaccination and spay/neuter fields are **owner-declared and not verified**. Say so in the app. Reactivity flags matter.
- **Terms:** a simple terms of service and community rules, 18+ only. Because real people meet in person, consider asking a lawyer to review the terms before launch even for a hobby project (this is a suggestion, not legal advice).
- **App stores later:** as I recall, Apple requires apps with user-generated content to include reporting and blocking. Building those in now avoids a rework. Verify when the phone app starts.

## 9. Risks

- **Cold start:** a matching app with few users is empty. Launch in one city, and consider a launch event or partnership with a dog park, rescue or training group.
- **Moderation workload** falls on the owner of the project.
- **Misuse as a human dating app.** Make the dog central to the profile and the rules.
- **Breeding is out of scope (decided: playdates only).** The intact filter stays, as a safety and preference setting for playdates. The terms and community rules should say the app is for playdates and does not support breeding or selling animals.
  - *If breeding is ever revisited:* it carries more risk than playdates (intact dogs meeting at parks, heat cycles, encouraging backyard breeding, health and legal questions). The earlier recommendation was an opt-in "looking for" setting with breeding off by default, optional health-test and registration fields marked unverified, and breeding conversations kept out of dog-park meetup suggestions. Colorado and local breeding rules have not been researched.
- **Liability** if a meetup goes badly (dog bite, harassment).

## 10. Rough cost

A small pilot is probably tens of dollars a month, mostly maps and email. This is an unverified estimate; price it properly when the architecture is chosen.

## 11. Suggested build order

1. ~~Static mock of the swipe screen, dog profile and questionnaire~~ **First draft built:** a clickable mockup in `mockup/` (open `mockup/index.html` in a browser). It uses sample dogs and parks, keeps data in the browser only, and includes the swipe screen, questionnaire, matching rules, chat, meetup planner and safety page. It has been click-tested in a simulated browser but not yet looked at in a real one or reviewed by the owner.
2. Sign-in and dog profiles with photos.
3. Discover and swipe with the hard filters.
4. Matching and chat.
5. Meetup planner and the Colorado Springs places list.
6. Safety tools and the admin view.
7. Soft launch with a small group of local dog owners.
8. Phone app, once the API has settled.

## 12. Open questions

- [ ] Name and branding. Working shortlist: Pikes Pack, Fetch Friends, Tail Mates. Availability (domain, app stores, trademark) is not checked.
- [x] Revenue: none for now (decided).
- [x] Default intact-dog filter for new users: spayed/neutered only (decided).
- [x] Temperament questions: approved for now (section 4).
- [ ] Build the first list of 15 to 30 Colorado Springs off-leash parks from the city page, and check the OSM license before using OSM data.
- [x] Hosting: AWS (decided).
- [ ] Frontend framework.
- [ ] Whether to get the terms of service reviewed before opening to the public.
- [ ] Who handles reports and moderation, and how fast.

## 13. How to resume

1. Read this file.
2. Confirm or change the open questions above, starting with the temperament questions and the intact-dog rule.
3. Start with step 1 of the build order: a clickable mock of the swipe screen, to show the girlfriend and a few friends.
4. Ask before creating any AWS resources or spending money.
