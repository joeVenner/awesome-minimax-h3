# MiniMax System Voice Catalogue

**332 preset voices across 24 language groups.** Verified 2026-08-04 by direct fetch of
`https://platform.minimax.io/docs/faq/system-voice-id.md`.

Pass any `Voice ID` below as `voice_setting.voice_id` on `/v1/t2a_v2`.

To list the live roster for an account — including cloned and designed voices — call:

```bash
curl -s -X POST "https://api.minimax.io/v1/get_voice" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{"voice_type": "all"}' | jq
```

`voice_type` ∈ `system` | `voice_cloning` | `voice_generation` | `all`.

> ⚠️ Copy IDs **exactly**. Several contain irregular spellings that are correct as published —
> e.g. `English_Trustworth_Man` (no trailing "y"), `French_Female_News Anchor` (contains a
> space). A typo returns `2013 invalid parameters`.

---

## Choosing a voice

The platform publishes **no audio descriptors** — the name is the only signal. Always audition
2–3 candidates with a representative sentence before committing to a long script.

### English narration shortlist

| Intent | Candidates |
|--------|-----------|
| Warm, low, documentary confidant | `English_Deep-VoicedGentleman`, `English_ManWithDeepVoice`, `English_magnetic_voiced_man` |
| Storytelling / trailer | `English_CaptivatingStoryteller`, `English_expressive_narrator`, `English_Steadymentor` |
| Calm female | `English_CalmWoman`, `English_SereneWoman`, `English_Graceful_Lady`, `English_Wiselady` |
| Trustworthy / corporate | `English_Trustworth_Man`, `English_PatientMan`, `English_ConfidentWoman` |
| Bright / upbeat | `English_Upbeat_Woman`, `English_radiant_girl`, `English_Jovialman` |
| Young / playful | `English_PlayfulGirl`, `English_WhimsicalGirl`, `English_LovelyGirl` |
| Authoritative | `English_MatureBoss`, `English_BossyLeader`, `English_AssertiveQueen`, `English_ImposingManner` |

Pair the voice with `language_boost` set to the matching language for best pronunciation.

---

## Complete list by language


### English (45)

| Voice ID | Name |
|---|---|
| `English_expressive_narrator` | Expressive Narrator |
| `English_radiant_girl` | Radiant Girl |
| `English_magnetic_voiced_man` | Magnetic-voiced Male |
| `English_compelling_lady1` | Compelling Lady |
| `English_Aussie_Bloke` | Aussie Bloke |
| `English_captivating_female1` | Captivating Female |
| `English_Upbeat_Woman` | Upbeat Woman |
| `English_Trustworth_Man` | Trustworthy Man |
| `English_CalmWoman` | Calm Woman |
| `English_UpsetGirl` | Upset Girl |
| `English_Gentle-voiced_man` | Gentle-voiced Man |
| `English_Whispering_girl` | Whispering girl |
| `English_Diligent_Man` | Diligent Man |
| `English_Graceful_Lady` | Graceful Lady |
| `English_ReservedYoungMan` | Reserved Young Man |
| `English_PlayfulGirl` | Playful Girl |
| `English_ManWithDeepVoice` | Man With Deep Voice |
| `English_MaturePartner` | Mature Partner |
| `English_FriendlyPerson` | Friendly Guy |
| `English_MatureBoss` | Bossy Lady |
| `English_Debator` | Male Debater |
| `English_LovelyGirl` | Lovely Girl |
| `English_Steadymentor` | Reliable Man |
| `English_Deep-VoicedGentleman` | Deep-voiced Gentleman |
| `English_Wiselady` | Wise Lady |
| `English_CaptivatingStoryteller` | Captivating Storyteller |
| `English_DecentYoungMan` | Decent Young Man |
| `English_SentimentalLady` | Sentimental Lady |
| `English_ImposingManner` | Imposing Queen |
| `English_SadTeen` | Teen Boy |
| `English_PassionateWarrior` | Passionate Warrior |
| `English_WiseScholar` | Wise Scholar |
| `English_Soft-spokenGirl` | Soft-Spoken Girl |
| `English_SereneWoman` | Serene Woman |
| `English_ConfidentWoman` | Confident Woman |
| `English_PatientMan` | Patient Man |
| `English_Comedian` | Comedian |
| `English_BossyLeader` | Bossy Leader |
| `English_Strong-WilledBoy` | Strong-Willed Boy |
| `English_StressedLady` | Stressed Lady |
| `English_AssertiveQueen` | Assertive Queen |
| `English_AnimeCharacter` | Female Narrator |
| `English_Jovialman` | Jovial Man |
| `English_WhimsicalGirl` | Whimsical Girl |
| `English_Kind-heartedGirl` | Kind-Hearted Girl |

### Chinese (Mandarin) (34)

| Voice ID | Name |
|---|---|
| `Chinese (Mandarin)_Reliable_Executive` | Reliable Executive |
| `Chinese (Mandarin)_News_Anchor` | News Anchor |
| `Chinese (Mandarin)_Unrestrained_Young_Man` | Unrestrained Young Man |
| `Chinese (Mandarin)_Mature_Woman` | Mature Woman |
| `Arrogant_Miss` | Arrogant Miss |
| `Robot_Armor` | Robot Armor |
| `Chinese (Mandarin)_Kind-hearted_Antie` | Kind-hearted Antie |
| `Chinese (Mandarin)_HK_Flight_Attendant` | HK Flight Attendant |
| `Chinese (Mandarin)_Humorous_Elder` | Humorous Elder |
| `Chinese (Mandarin)_Gentleman` | Gentleman |
| `Chinese (Mandarin)_Warm_Bestie` | Warm Bestie |
| `Chinese (Mandarin)_Stubborn_Friend` | Stubborn Friend |
| `Chinese (Mandarin)_Sweet_Lady` | Sweet Lady |
| `Chinese (Mandarin)_Southern_Young_Man` | Southern Young Man |
| `Chinese (Mandarin)_Wise_Women` | Wise Women |
| `Chinese (Mandarin)_Gentle_Youth` | Gentle Youth |
| `Chinese (Mandarin)_Warm_Girl` | Warm Girl |
| `Chinese (Mandarin)_Male_Announcer` | Male Announcer |
| `Chinese (Mandarin)_Kind-hearted_Elder` | Kind-hearted Elder |
| `Chinese (Mandarin)_Cute_Spirit` | Cute Spirit |
| `Chinese (Mandarin)_Radio_Host` | Radio Host |
| `Chinese (Mandarin)_Lyrical_Voice` | Lyrical Voice |
| `Chinese (Mandarin)_Straightforward_Boy` | Straightforward Boy |
| `Chinese (Mandarin)_Sincere_Adult` | Sincere Adult |
| `Chinese (Mandarin)_Gentle_Senior` | Gentle Senior |
| `Chinese (Mandarin)_Crisp_Girl` | Crisp Girl |
| `Chinese (Mandarin)_Pure-hearted_Boy` | Pure-hearted Boy |
| `Chinese (Mandarin)_Soft_Girl` | Soft Girl |
| `Chinese (Mandarin)_IntellectualGirl` | Intellectual Girl |
| `Chinese (Mandarin)_Warm_HeartedGirl` | Warm-hearted Girl |
| `Chinese (Mandarin)_Laid_BackGirl` | Laid-back Girl |
| `Chinese (Mandarin)_ExplorativeGirl` | Explorative Girl |
| `Chinese (Mandarin)_Warm-HeartedAunt` | Warm-hearted Aunt |
| `Chinese (Mandarin)_BashfulGirl` | Bashful Girl |

### Japanese (15)

| Voice ID | Name |
|---|---|
| `Japanese_IntellectualSenior` | Intellectual Senior |
| `Japanese_DecisivePrincess` | Decisive Princess |
| `Japanese_LoyalKnight` | Loyal Knight |
| `Japanese_DominantMan` | Dominant Man |
| `Japanese_SeriousCommander` | Serious Commander |
| `Japanese_ColdQueen` | Cold Queen |
| `Japanese_DependableWoman` | Dependable Woman |
| `Japanese_GentleButler` | Gentle Butler |
| `Japanese_KindLady` | Kind Lady |
| `Japanese_CalmLady` | Calm Lady |
| `Japanese_OptimisticYouth` | Optimistic Youth |
| `Japanese_GenerousIzakayaOwner` | Generous Izakaya Owner |
| `Japanese_SportyStudent` | Sporty Student |
| `Japanese_InnocentBoy` | Innocent Boy |
| `Japanese_GracefulMaiden` | Graceful Maiden |

### Cantonese (6)

| Voice ID | Name |
|---|---|
| `Cantonese_ProfessionalHost (F)` | Professional Female Host |
| `Cantonese_GentleLady` | Gentle Lady |
| `Cantonese_ProfessionalHost (M)` | Professional Male Host |
| `Cantonese_PlayfulMan` | Playful Man |
| `Cantonese_CuteGirl` | Cute Girl |
| `Cantonese_KindWoman` | Kind Woman |

### Korean (49)

| Voice ID | Name |
|---|---|
| `Korean_AirheadedGirl` | Airheaded Girl |
| `Korean_AthleticGirl` | Athletic Girl |
| `Korean_AthleticStudent` | Athletic Student |
| `Korean_BraveAdventurer` | Brave Adventurer |
| `Korean_BraveFemaleWarrior` | Brave Female Warrior |
| `Korean_BraveYouth` | Brave Youth |
| `Korean_CalmGentleman` | Calm Gentleman |
| `Korean_CalmLady` | Calm Lady |
| `Korean_CaringWoman` | Caring Woman |
| `Korean_CharmingElderSister` | Charming Elder Sister |
| `Korean_CharmingSister` | Charming Sister |
| `Korean_CheerfulBoyfriend` | Cheerful Boyfriend |
| `Korean_CheerfulCoolJunior` | Cheerful Cool Junior |
| `Korean_CheerfulLittleSister` | Cheerful Little Sister |
| `Korean_ChildhoodFriendGirl` | Childhood Friend Girl |
| `Korean_CockyGuy` | Cocky Guy |
| `Korean_ColdGirl` | Cold Girl |
| `Korean_ColdYoungMan` | Cold Young Man |
| `Korean_ConfidentBoss` | Confident Boss |
| `Korean_ConsiderateSenior` | Considerate Senior |
| `Korean_DecisiveQueen` | Decisive Queen |
| `Korean_DominantMan` | Dominant Man |
| `Korean_ElegantPrincess` | Elegant Princess |
| `Korean_EnchantingSister` | Enchanting Sister |
| `Korean_EnthusiasticTeen` | Enthusiastic Teen |
| `Korean_FriendlyBigSister` | Friendly Big Sister |
| `Korean_GentleBoss` | Gentle Boss |
| `Korean_GentleWoman` | Gentle Woman |
| `Korean_HaughtyLady` | Haughty Lady |
| `Korean_InnocentBoy` | Innocent Boy |
| `Korean_IntellectualMan` | Intellectual Man |
| `Korean_IntellectualSenior` | Intellectual Senior |
| `Korean_LonelyWarrior` | Lonely Warrior |
| `Korean_MatureLady` | Mature Lady |
| `Korean_MysteriousGirl` | Mysterious Girl |
| `Korean_OptimisticYouth` | Optimistic Youth |
| `Korean_PlayboyCharmer` | Playboy Charmer |
| `Korean_PossessiveMan` | Possessive Man |
| `Korean_QuirkyGirl` | Quirky Girl |
| `Korean_ReliableSister` | Reliable Sister |
| `Korean_ReliableYouth` | Reliable Youth |
| `Korean_SassyGirl` | Sassy Girl |
| `Korean_ShyGirl` | Shy Girl |
| `Korean_SoothingLady` | Soothing Lady |
| `Korean_StrictBoss` | Strict Boss |
| `Korean_SweetGirl` | Sweet Girl |
| `Korean_ThoughtfulWoman` | Thoughtful Woman |
| `Korean_WiseElf` | Wise Elf |
| `Korean_WiseTeacher` | Wise Teacher |

### Spanish (47)

| Voice ID | Name |
|---|---|
| `Spanish_SereneWoman` | Serene Woman |
| `Spanish_MaturePartner` | Mature Partner |
| `Spanish_CaptivatingStoryteller` | Captivating Storyteller |
| `Spanish_Narrator` | Narrator |
| `Spanish_WiseScholar` | Wise Scholar |
| `Spanish_Kind-heartedGirl` | Kind-hearted Girl |
| `Spanish_DeterminedManager` | Determined Manager |
| `Spanish_BossyLeader` | Bossy Leader |
| `Spanish_ReservedYoungMan` | Reserved Young Man |
| `Spanish_ConfidentWoman` | Confident Woman |
| `Spanish_ThoughtfulMan` | Thoughtful Man |
| `Spanish_Strong-WilledBoy` | Strong-willed Boy |
| `Spanish_SophisticatedLady` | Sophisticated Lady |
| `Spanish_RationalMan` | Rational Man |
| `Spanish_AnimeCharacter` | Anime Character |
| `Spanish_Deep-tonedMan` | Deep-toned Man |
| `Spanish_Fussyhostess` | Fussy hostess |
| `Spanish_SincereTeen` | Sincere Teen |
| `Spanish_FrankLady` | Frank Lady |
| `Spanish_Comedian` | Comedian |
| `Spanish_Debator` | Debator |
| `Spanish_ToughBoss` | Tough Boss |
| `Spanish_Wiselady` | Wise Lady |
| `Spanish_Steadymentor` | Steady Mentor |
| `Spanish_Jovialman` | Jovial Man |
| `Spanish_SantaClaus` | Santa Claus |
| `Spanish_Rudolph` | Rudolph |
| `Spanish_Intonategirl` | Intonate Girl |
| `Spanish_Arnold` | Arnold |
| `Spanish_Ghost` | Ghost |
| `Spanish_HumorousElder` | Humorous Elder |
| `Spanish_EnergeticBoy` | Energetic Boy |
| `Spanish_WhimsicalGirl` | Whimsical Girl |
| `Spanish_StrictBoss` | Strict Boss |
| `Spanish_ReliableMan` | Reliable Man |
| `Spanish_SereneElder` | Serene Elder |
| `Spanish_AngryMan` | Angry Man |
| `Spanish_AssertiveQueen` | Assertive Queen |
| `Spanish_CaringGirlfriend` | Caring Girlfriend |
| `Spanish_PowerfulSoldier` | Powerful Soldier |
| `Spanish_PassionateWarrior` | Passionate Warrior |
| `Spanish_ChattyGirl` | Chatty Girl |
| `Spanish_RomanticHusband` | Romantic Husband |
| `Spanish_CompellingGirl` | Compelling Girl |
| `Spanish_PowerfulVeteran` | Powerful Veteran |
| `Spanish_SensibleManager` | Sensible Manager |
| `Spanish_ThoughtfulLady` | Thoughtful Lady |

### Portuguese (73)

| Voice ID | Name |
|---|---|
| `Portuguese_SentimentalLady` | Sentimental Lady |
| `Portuguese_BossyLeader` | Bossy Leader |
| `Portuguese_Wiselady` | Wise lady |
| `Portuguese_Strong-WilledBoy` | Strong-willed Boy |
| `Portuguese_Deep-VoicedGentleman` | Deep-voiced Gentleman |
| `Portuguese_UpsetGirl` | Upset Girl |
| `Portuguese_PassionateWarrior` | Passionate Warrior |
| `Portuguese_AnimeCharacter` | Anime Character |
| `Portuguese_ConfidentWoman` | Confident Woman |
| `Portuguese_AngryMan` | Angry Man |
| `Portuguese_CaptivatingStoryteller` | Captivating Storyteller |
| `Portuguese_Godfather` | Godfather |
| `Portuguese_ReservedYoungMan` | Reserved Young Man |
| `Portuguese_SmartYoungGirl` | Smart Young Girl |
| `Portuguese_Kind-heartedGirl` | Kind-hearted Girl |
| `Portuguese_Pompouslady` | Pompous lady |
| `Portuguese_Grinch` | Grinch |
| `Portuguese_Debator` | Debator |
| `Portuguese_SweetGirl` | Sweet Girl |
| `Portuguese_AttractiveGirl` | Attractive Girl |
| `Portuguese_ThoughtfulMan` | Thoughtful Man |
| `Portuguese_PlayfulGirl` | Playful Girl |
| `Portuguese_GorgeousLady` | Gorgeous Lady |
| `Portuguese_LovelyLady` | Lovely Lady |
| `Portuguese_SereneWoman` | Serene Woman |
| `Portuguese_SadTeen` | Sad Teen |
| `Portuguese_MaturePartner` | Mature Partner |
| `Portuguese_Comedian` | Comedian |
| `Portuguese_NaughtySchoolgirl` | Naughty Schoolgirl |
| `Portuguese_Narrator` | Narrator |
| `Portuguese_ToughBoss` | Tough Boss |
| `Portuguese_Fussyhostess` | Fussy hostess |
| `Portuguese_Dramatist` | Dramatist |
| `Portuguese_Steadymentor` | Steady Mentor |
| `Portuguese_Jovialman` | Jovial Man |
| `Portuguese_CharmingQueen` | Charming Queen |
| `Portuguese_SantaClaus` | Santa Claus |
| `Portuguese_Rudolph` | Rudolph |
| `Portuguese_Arnold` | Arnold |
| `Portuguese_CharmingSanta` | Charming Santa |
| `Portuguese_CharmingLady` | Charming Lady |
| `Portuguese_Ghost` | Ghost |
| `Portuguese_HumorousElder` | Humorous Elder |
| `Portuguese_CalmLeader` | Calm Leader |
| `Portuguese_GentleTeacher` | Gentle Teacher |
| `Portuguese_EnergeticBoy` | Energetic Boy |
| `Portuguese_ReliableMan` | Reliable Man |
| `Portuguese_SereneElder` | Serene Elder |
| `Portuguese_GrimReaper` | Grim Reaper |
| `Portuguese_AssertiveQueen` | Assertive Queen |
| `Portuguese_WhimsicalGirl` | Whimsical Girl |
| `Portuguese_StressedLady` | Stressed Lady |
| `Portuguese_FriendlyNeighbor` | Friendly Neighbor |
| `Portuguese_CaringGirlfriend` | Caring Girlfriend |
| `Portuguese_PowerfulSoldier` | Powerful Soldier |
| `Portuguese_FascinatingBoy` | Fascinating Boy |
| `Portuguese_RomanticHusband` | Romantic Husband |
| `Portuguese_StrictBoss` | Strict Boss |
| `Portuguese_InspiringLady` | Inspiring Lady |
| `Portuguese_PlayfulSpirit` | Playful Spirit |
| `Portuguese_ElegantGirl` | Elegant Girl |
| `Portuguese_CompellingGirl` | Compelling Girl |
| `Portuguese_PowerfulVeteran` | Powerful Veteran |
| `Portuguese_SensibleManager` | Sensible Manager |
| `Portuguese_ThoughtfulLady` | Thoughtful Lady |
| `Portuguese_TheatricalActor` | Theatrical Actor |
| `Portuguese_FragileBoy` | Fragile Boy |
| `Portuguese_ChattyGirl` | Chatty Girl |
| `Portuguese_Conscientiousinstructor` | Conscientious Instructor |
| `Portuguese_RationalMan` | Rational Man |
| `Portuguese_WiseScholar` | Wise Scholar |
| `Portuguese_FrankLady` | Frank Lady |
| `Portuguese_DeterminedManager` | Determined Manager |

### French (6)

| Voice ID | Name |
|---|---|
| `French_Male_Speech_New` | Level-Headed Man |
| `French_Female_News Anchor` | Patient Female Presenter |
| `French_CasualMan` | Casual Man |
| `French_MovieLeadFemale` | Movie Lead Female |
| `French_FemaleAnchor` | Female Anchor |
| `French_MaleNarrator` | Male Narrator |

### Indonesian (9)

| Voice ID | Name |
|---|---|
| `Indonesian_SweetGirl` | Sweet Girl |
| `Indonesian_ReservedYoungMan` | Reserved Young Man |
| `Indonesian_CharmingGirl` | Charming Girl |
| `Indonesian_CalmWoman` | Calm Woman |
| `Indonesian_ConfidentWoman` | Confident Woman |
| `Indonesian_CaringMan` | Caring Man |
| `Indonesian_BossyLeader` | Bossy Leader |
| `Indonesian_DeterminedBoy` | Determined Boy |
| `Indonesian_GentleGirl` | Gentle Girl |

### German (3)

| Voice ID | Name |
|---|---|
| `German_FriendlyMan` | Friendly Man |
| `German_SweetLady` | Sweet Lady |
| `German_PlayfulMan` | Playful Man |

### Russian (8)

| Voice ID | Name |
|---|---|
| `Russian_HandsomeChildhoodFriend` | Handsome Childhood Friend |
| `Russian_BrightHeroine` | Bright Queen |
| `Russian_AmbitiousWoman` | Ambitious Woman |
| `Russian_ReliableMan` | Reliable Man |
| `Russian_CrazyQueen` | Crazy Girl |
| `Russian_PessimisticGirl` | Pessimistic Girl |
| `Russian_AttractiveGuy` | Attractive Guy |
| `Russian_Bad-temperedBoy` | Bad-tempered Boy |

### Italian (4)

| Voice ID | Name |
|---|---|
| `Italian_BraveHeroine` | Brave Heroine |
| `Italian_Narrator` | Narrator |
| `Italian_WanderingSorcerer` | Wandering Sorcerer |
| `Italian_DiligentLeader` | Diligent Leader |

### Dutch (2)

| Voice ID | Name |
|---|---|
| `Dutch_kindhearted_girl` | Kind-hearted girl |
| `Dutch_bossy_leader` | Bossy leader |

### Vietnamese (1)

| Voice ID | Name |
|---|---|
| `Vietnamese_kindhearted_girl` | Kind-hearted girl |

### Arabic (2)

| Voice ID | Name |
|---|---|
| `Arabic_CalmWoman` | Calm Woman |
| `Arabic_FriendlyGuy` | Friendly Guy |

### Turkish (2)

| Voice ID | Name |
|---|---|
| `Turkish_CalmWoman` | Calm Woman |
| `Turkish_Trustworthyman` | Trustworthy man |

### Ukrainian (2)

| Voice ID | Name |
|---|---|
| `Ukrainian_CalmWoman` | Calm Woman |
| `Ukrainian_WiseScholar` | Wise Scholar |

### Thai (4)

| Voice ID | Name |
|---|---|
| `Thai_male_1_sample8` | Serene Man |
| `Thai_male_2_sample2` | Friendly Man |
| `Thai_female_1_sample1` | Confident Woman |
| `Thai_female_2_sample2` | Energetic Woman |

### Polish (4)

| Voice ID | Name |
|---|---|
| `Polish_male_1_sample4` | Male Narrator |
| `Polish_male_2_sample3` | Male Anchor |
| `Polish_female_1_sample1` | Calm Woman |
| `Polish_female_2_sample3` | Casual Woman |

### Romanian (4)

| Voice ID | Name |
|---|---|
| `Romanian_male_1_sample2` | Reliable Man |
| `Romanian_male_2_sample1` | Energetic Youth |
| `Romanian_female_1_sample4` | Optimistic Youth |
| `Romanian_female_2_sample1` | Gentle Woman |

### Greek (3)

| Voice ID | Name |
|---|---|
| `greek_male_1a_v1` | Thoughtful Mentor |
| `Greek_female_1_sample1` | Gentle Lady |
| `Greek_female_2_sample3` | Girl Next Door |

### Czech (3)

| Voice ID | Name |
|---|---|
| `czech_male_1_v1` | Assured Presenter |
| `czech_female_5_v7` | Steadfast Narrator |
| `czech_female_2_v2` | Elegant Lady |

### Finnish (3)

| Voice ID | Name |
|---|---|
| `finnish_male_3_v1` | Upbeat Man |
| `finnish_male_1_v2` | Friendly Boy |
| `finnish_female_4_v1` | Assetive Woman |

### Hindi (3)

| Voice ID | Name |
|---|---|
| `hindi_male_1_v2` | Trustworthy Advisor |
| `hindi_female_2_v1` | Tranquil Woman |
| `hindi_female_1_v2` | News Anchor |
