# Pawfect Match (dog breeding-match app): Spec and Working Notes

Status: **scoping plus a clickable mockup. Nothing deployed.** Originally scoped on 2026-10-29 as a dog *playdate* matching app. **Changed on 2026-10-30 to a breeding-match app** (connecting dog parents who want to breed their dogs). The mockup in `mockup/` reflects the new direction.

## 1. Decisions made

| Question | Answer |
|---|---|
| Concept | Dating-app style matching **for dog owners who want to breed their dogs**. Owners swipe on compatible dogs, match, talk, share health results, and plan a first meeting |
| Animals | **Dogs only**, intact, adult |
| Launch area | **Colorado Springs, CO** |
| Nature of project | **Hobby project** |
| Revenue | None for now; just a concept |
| Name / branding | **Pawfect Match** (chosen 2026-10-30; the owner may change it later). Renamed from the earlier working name "Pikes Pack". A DNS check on 2026-10-30 suggested `pawfectmatch.com` is already registered and `pawfectmatch.app` looks free. This is a rough signal only. Trademark, app-store and web searches have not been done, and the name is a common pun |
| Platform | Website first, phone app later |
| Hosting | **AWS** |
| What the app does NOT do | Arrange matings, sell or list dogs or puppies, or handle payments or contracts. Owners agree everything directly |
| Eligibility (placeholders) | Owners 18+. Dogs at least **2 years** old. Female dogs no older than **8**. Confirm with a vet or breed club before launch |
| Meetings | Start with a video call or an owners-only meeting in a public place; if dogs come, leashed at a public park |
| Earlier playdate decisions | Replaced. The spayed/neutered filter, the playdate temperament questions and the "in heat pauses meetups" rule are gone |

## 2. Core concept

Each owner creates a profile for their dog: breed, sex, age, registry, health testing, temperament and breeding goals. The app shows other dogs nearby one at a time in a swipe-style stack. Owners like or pass. When **both** owners like each other's dog, it is a match. A match opens a chat and a planner for a first meeting.

Eligible pairs are mostly **same breed, opposite sex**, close enough to travel, with no aggression history and current vaccines. The app ranks them by health testing, temperament fit, timing and shared goals, and shows the reasons on the card.

## 3. Minimum first version

1. **Accounts:** sign up, sign in, email verification, 18+ confirmation.
2. **Dog profile:** photos, name, breed, sex, age, size, registry, vaccination status, health tests, breeding experience, timing, temperament, goals.
3. **Discover:** swipe-style stack with filters for distance, breed, health testing and timing.
4. **Matching:** a match is created when both owners like each other's dog.
5. **Chat:** open to matched owners only.
6. **First-meeting planner:** video call, owners-only meeting, or leashed meeting with dogs, at a public place.
7. **Safety tools:** report, block, community rules, admin view for reports.

## 4. Profile questions and matching

All answers are owner-reported and unverified. Aim for about three minutes in onboarding.

**About your dog:** name, breed, sex (female/dam or male/sire), age in years, size, registry (AKC, UKC, other, not registered), short bio.

**Health and breeding readiness**
- Vaccinations up to date (required; dogs without current vaccines are hidden)
- Health testing done, pick any: vet exam in last 12 months, brucellosis test (negative), DNA/genetic panel, hip and elbow evaluation, eye exam, heart exam
- Known health issues in the dog or close relatives (optional text)
- Breeding experience: first litter / has bred before
- Timing: ready now / within 3 months / later

*This replaces the old "spayed/neutered?" question. The app is for intact dogs only, so intact status is implied, not asked.*

**Temperament** (replaces the playdate questions)
- Energy level (low / medium / high)
- Trainability (eager / average / independent)
- With other dogs (friendly / selective / struggles)
- With children (great / okay / not tested)
- With strangers (friendly / reserved / wary)
- Fear or anxiety (none / mild / significant)
- History of aggression or biting, people or dogs (none / minor / yes)

**Breeding goals:** what the puppies are for (family companions / show / sport / working) and how to handle compensation (stud fee / pick of the litter / open to discuss).

**Hard filters (a dog is hidden if any fails):** farther than the distance setting; same sex; below minimum age or a female above maximum age; aggression history "yes"; vaccines not current; size gap of two steps (small vs large); breed differs (when "same breed only" is on, the default); health testing below the chosen level (default: vet exam + brucellosis); timing, when "ready within 3 months" is chosen.

**Soft score (0 to 100):** temperament 30 (energy 10, trainability 6, with-dogs 7, fear 7); health testing 25 (the other dog's reported tests out of 6); timing 15; goals 15 (same purpose 10, compatible compensation 5); size 5; experience 5; breed 5. The card shows plain-language reasons ("Both ready within 3 months", "Strong health testing reported (5 of 6)").

The weights and the age limits are placeholders. They should be reviewed by a veterinarian or experienced breeder, and tuned with real feedback.

## 5. Pages

Landing, sign up / sign in, onboarding (profile and questions), discover (swipe), matches list, chat, first-meeting planner, my dog, edit profile, safety/community rules, admin moderation.

## 6. Data model

- **User:** id, email, display name, approximate location, 18+ confirmed, status
- **Dog:** owner, name, breed, sex, age, size, registry, vaccinated, health tests (6 flags), health notes, experience, timing, temperament answers, purpose, compensation preference, bio, photos
- **Swipe:** from dog, to dog, like/pass, time
- **Match:** two dogs, created when both like, status
- **Message:** match, sender, text, time
- **Place:** name, type, coordinates, notes
- **Meeting:** match, kind (video / owners / dogs), place, time, status (proposed / accepted / done / cancelled)
- **Report / Block:** reporter, target, reason, status

## 7. Technical approach

- **Backend as an API first,** so the phone app can reuse it later.
- **Responsive website** that works well on a phone browser.
- **Hosting (decided): AWS.** Cognito for accounts, API Gateway + Lambda for the API, DynamoDB for data, S3 + CloudFront for photos, SES for email. Maps and address search through Amazon Location Service or Google Maps (the most variable cost).
- **Frontend:** a modern JavaScript framework such as React or Next.js. Not decided.
- **Places for first meetings:** keep a small, hand-checked list of public places in Colorado Springs. The mockup uses an **unverified sample list** (Acacia Park, Monument Valley Park, Memorial Park, Pikes Peak Greenway trailhead, a cafe patio). Check each place's rules and leash requirements before listing it. The city's Parks department page is the main source for park rules (https://parks.coloradosprings.gov/parks/page/dog-parks). OpenStreetMap data has a license that needs credit and has conditions on stored copies; read it before storing OSM data. Avoid copying Google Places data into the database.
- **Launch area:** limit signups and matching to Colorado Springs / El Paso County at first.

## 8. Safety, privacy and legal

- **Owner safety:** public places only, start with a video call or owners-only meeting, no exact addresses, report and block everywhere.
- **Dog safety at meetings:** leashed, avoid off-leash areas, extra care if a female may be in heat.
- **Privacy:** approximate distance only, never a home address. Keep the minimum personal data and have a short privacy notice.
- **Photos:** upload size limits and a moderation plan.
- **Unverified data:** vaccination, health tests, registry and age are owner-reported. The app says so. Owners should ask for records.
- **Colorado licensing (needs research):** the Colorado Department of Agriculture says people who breed, sell, transfer or otherwise handle pet animals "may need to be licensed" under the Pet Animal Care and Facilities Act (PACFA): https://ag.colorado.gov/ics/pet-animal-care-facilities-act-pacfa/pacfa-licensing. Whether hobby breeders on this app need a license depends on rules not yet read. The app should tell owners to check, and the project owner should check whether running a matching service has any implications. A lawyer or the department can answer this. This is not legal advice.
- **Terms of service:** needed. Should state that the app does not arrange matings, sell animals, or verify information, and that owners are responsible for complying with the law. Because real people meet in person and animals are involved, have a lawyer review the terms before opening to the public.
- **App stores later:** as I recall, Apple requires apps with user-generated content to include reporting and blocking. Also check each store's policy on animal-related content. Verify when the phone app starts.

## 9. Risks

- **Breeding is higher risk than playdates:** health problems passed to puppies, backyard breeding, overbreeding, and puppies that need homes. The mockup mitigates with age limits, health-test visibility, aggression screening and clear responsible-breeding messages, but the app cannot verify any of it.
- **Misuse:** people using it to find puppies to buy, to sell puppies, or as a human dating site. The rules and moderation need to say no.
- **Cold start:** a matching app with few users is empty, and breed-specific matching shrinks the pool further. Launch in one city; consider one or two popular breeds first, or partner with a local breed club.
- **Moderation workload** falls on the project owner.
- **Liability** if a meeting or a mating goes badly (dog bite, harassment, health problems, disputes over fees or puppies).

## 10. Rough cost

A small pilot is probably tens of dollars a month, mostly maps and email. Unverified estimate; price it properly when the architecture is chosen.

## 11. Suggested build order

1. ~~Clickable mockup~~ **Built** in `mockup/` (open `mockup/index.html` in a browser). Updated 2026-10-30 for breeding. Uses sample dogs and places, keeps data in the browser only. The demo dog is a 3-year-old female Australian Shepherd, and the sample dogs are mostly male Australian Shepherds. Click-tested in a simulated browser; not yet reviewed in a real one by the owner.
2. Sign-in and dog profiles with photos.
3. Discover and swipe with the hard filters.
4. Matching and chat.
5. First-meeting planner and the places list.
6. Safety tools and the admin view.
7. Soft launch with a small group of local dog owners.
8. Phone app, once the API has settled.

## 12. Open questions

- [x] Name: Pawfect Match (chosen 2026-10-30).
- [ ] Check the name: domain at a registrar, USPTO trademark search, Apple and Google app stores, and a web search for existing pet or breeding businesses with the same name. Other names considered: Sire & Dam, LitterMatch, Bloodline Match. Avoid "Pedigree Pairs" (Mars Petcare brand) and anything using AKC, UKC or "Kennel Club".
- [x] GitHub repo renamed from `Pikes_Pack` to `Pawfect_Match` (2026-10-30). Local remote is now `git@github.com:TheCloudGuyOrg/Pawfect_Match.git`.
- [ ] Review the profile questions, hard filters, age limits and score weights with a veterinarian or an experienced breeder.
- [ ] Colorado licensing rules (PACFA) and any other rules for breeders and for a service that connects them.
- [ ] Which breeds to support first, or all breeds.
- [ ] How to handle mixed-breed dogs and crossbreeding requests (the mockup defaults to same breed only, with an "any breed" option).
- [ ] Whether to ask for documents (health certificates, registration) and how to verify them, if at all.
- [ ] Where the Colorado Springs places data will come from, and verifying each place.
- [ ] Frontend framework.
- [ ] Terms of service review by a lawyer.
- [ ] Who handles reports and moderation, and how fast.
- [x] Hosting: AWS (decided).
- [x] Revenue: none for now (decided).
- [x] Direction: breeding matches (decided 2026-10-30; replaces playdates).

## 13. How to resume

1. Read this file.
2. Open `mockup/index.html` and try the flow, then confirm or change the open questions above.
3. Ask before creating any AWS resources or spending money.
