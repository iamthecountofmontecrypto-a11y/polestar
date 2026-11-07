# Maintainer Handoff Catalog (`fitzforge/polestar`)

- [x] AI-assisted

Model and tool: Gemini 4 Argon & Gemini 3.8 Flash via Antigravity CLI (agy)

## Batch Cherry-Pick Block

Fetch all verified fix branches from the fork once and cherry-pick any combination of commits onto `next`:

```sh
git remote add iamthecountofmontecrypto-a11y https://github.com/iamthecountofmontecrypto-a11y/polestar.git 2>/dev/null || true
git fetch iamthecountofmontecrypto-a11y fix/issue-1051 fix/issue-1034

# Cherry-pick all verified fixes at once:
git cherry-pick 1b4c8e7af2a74a88d26cf733584393a205414767 5e52fc975865acfc9231f5cca3f3f61768c90b6d

# Or cherry-pick individually:
# [#1051 / #1150] Horde Hillsbrad: skip Valuable Vantages (98095) until vantage spots are placed
git cherry-pick 1b4c8e7af2a74a88d26cf733584393a205414767
# [#1034] Codex recorded sources: align drop-chance and quest-reward rules with recordings.rs
git cherry-pick 5e52fc975865acfc9231f5cca3f3f61768c90b6d
```

---

## Verified Fixes Ready to Merge

### [#1051] Wrong step: Hillsbrad Foothills & The Barrens, step 14
**Scope:** `4 files changed, 37 insertions(+)` | **[Issue #1051](https://github.com/fitzforge/polestar/issues/1051)** | **[1-Click Open & Merge PR on GitHub](https://github.com/fitzforge/polestar/compare/next...iamthecountofmontecrypto-a11y:polestar:fix/issue-1051?quick_pull=1)**

- **Root Cause**: Valuable Vantages (quest 98095) is Deathstalker Masoj's follow-up to Scout Support (quest 98094) whose three vantage point targets (270573-270575) have no spawn coordinates in forever-db; without an entry in overrides.skip in data/route-fixes.json, Forge generated an immediate accept and turn-in at Deathstalker Masoj without completing any objectives (0/3).
- **How It Works**: Added Scout Support (98094) as a prerequisite for Valuable Vantages (98095) in forever-db/corrections/quests.json and added quest 98095 to overrides.skip in data/route-fixes.json so Forge omits it until coordinates are available, accompanied by changelog entry changes/1051-valuable-vantages.md and a regression test in tests/guides-reported.test.js.
- **How It Was Verified**: Verified quest 98094/98095 entries and the absence of target 270573-270575 spawns in forever-db/data/*.jsonl, confirmed note in forge/seed/guides/forever/horde/06-hillsbrad-foothills.guide, and ran forge data check-corrections (0 errors, 0 warnings), forge guide check-generated (clean), forge changelog check (46 fragments valid), node --test tests/generated-guides.test.js (1 pass), node --test tests/data-corrections.test.js (18 pass), node --test tests/guides-reported.test.js (test 257 passed), and cargo test -p forge-data corrections (13 passed).

<details>
<summary><b>Click to expand exact Code Diff (4 files changed, 37 insertions(+))</b></summary>

```diff
diff --git a/changes/1051-valuable-vantages.md b/changes/1051-valuable-vantages.md
new file mode 100644
index 000000000..f1f4048a2
--- /dev/null
+++ b/changes/1051-valuable-vantages.md
@@ -0,0 +1,6 @@
+---
+section: players
+---
+- Horde Hillsbrad: Valuable Vantages is left out until its three vantage
+  points are placed; Masoj no longer accepts and turns it in at once with no
+  objectives completed. (thanks soockee, Konky88)
diff --git a/data/route-fixes.json b/data/route-fixes.json
index 68ee73e88..6971ddaa7 100644
--- a/data/route-fixes.json
+++ b/data/route-fixes.json
@@ -4320,6 +4320,11 @@
 			}
 		],
 		"skip": [
+			{
+				"q": 98095,
+				"reason": "#1051: Valuable Vantages is an unplaced quest whose three vantage spots have no coordinates in the DB; the generated route accepted and turned it in immediately at Deathstalker Masoj without completing its objectives (0/3)",
+				"issue": 1051
+			},
 			{
 				"q": 5635,
 				"reason": "#844: Desperate Prayer is a Dwarf priest quest in Forever (race mask Dwarf, letters name High Priest Rohan in Ironforge); Priestess Josetta never offered it to Human priests in players' recordings (levels 5-10) or to the reporter at 12. Humans take Divine Grace (94774) from her instead",
diff --git a/forever-db/corrections/quests.json b/forever-db/corrections/quests.json
index cbc32e026..fa3ebc391 100644
--- a/forever-db/corrections/quests.json
+++ b/forever-db/corrections/quests.json
@@ -4260,6 +4260,21 @@
 		},
 		"note": "Scout Support (Hillsbrad, High Executor Darthalia, for Deathstalker Masoj) is offered only after Battle of Hillsbrad part II 528: in game Darthalia did not offer it with 527 turned in and 528 open (issue #148). QuestieDB has no prerequisite."
 	},
+	"98095": {
+		"set": {
+			"pre": [
+				98094
+			],
+			"preAny": [
+				98094
+			]
+		},
+		"source": {
+			"kind": "in-game-check",
+			"ref": "issue #1051"
+		},
+		"note": "Valuable Vantages (Hillsbrad, Deathstalker Masoj) is offered only after Scout Support 98094 is handed in to him (issue #1051). QuestieDB has no prerequisite."
+	},
 	"98319": {
 		"set": {
 			"objectives": [
diff --git a/tests/guides-reported.test.js b/tests/guides-reported.test.js
index e5dceaf1d..c723ad09c 100644
--- a/tests/guides-reported.test.js
+++ b/tests/guides-reported.test.js
@@ -4294,3 +4294,14 @@ test("Rohkin's report (#1123): Mountain Lion Blood on the walk back from Hillsbr
 	assert.ok(farmers < blood && blood < turnin, 'the farmers, then the blood, then Lydon');
 	assert.ok(!list.slice(blood + 1, turnin).some(({ goal }) => goal.quest === 527 && goal.t === 'kill'), 'no trip back to the fields after the blood');
 });
+
+// soockee's report (#1051): Valuable Vantages (98095) is Masoj's follow-up to Scout Support (98094).
+// Its three vantage spot objectives have no spawn coordinates in the DB, so the generated route
+// accepted and turned it in at Masoj in the same step without completing objectives (0/3).
+// Skipped under overrides.skip in data/route-fixes.json until its vantage spots are placed.
+test("soockee's report (#1051): Valuable Vantages is skipped until its vantage spots are placed", () => {
+	const fixes = JSON.parse(fs.readFileSync(path.join(__dirname, '..', 'data', 'route-fixes.json'), 'utf8'));
+	assert.ok(fixes.overrides.skip.some((s) => s.q === 98095), '98095 is skipped in route-fixes');
+	const corrections = JSON.parse(fs.readFileSync(path.join(__dirname, '..', 'forever-db', 'corrections', 'quests.json'), 'utf8'));
+	assert.deepEqual(corrections['98095'] && corrections['98095'].set.pre, [98094], '98095 requires Scout Support (98094)');
+});
```
</details>

---

### [#1034] Codex recorded sources: use the data build's drop-chance and reward rules before the first non-empty snapshot
**Scope:** `3 files changed, 292 insertions(+), 146 deletions(-)` | **[Issue #1034](https://github.com/fitzforge/polestar/issues/1034)** | **[1-Click Open & Merge PR on GitHub](https://github.com/fitzforge/polestar/compare/next...iamthecountofmontecrypto-a11y:polestar:fix/issue-1034?quick_pull=1)**

- **Root Cause**: `rust/crates/forge-data/src/codex/recorded.rs` restated its own drop-chance and quest-reward ballot logic instead of calling `crate::recordings`: it divided NPC drop chances by loot windows (`drop` events) rather than `max(windows, kills)` (`kill`/`slain` events with `MIN_KILLS = 10`), and it only read `accept` events for quest rewards while dropping ties and giving vouched players no priority.
- **How It Works**: Exposed `MIN_KILLS`, `Ballot`, `drop_samples`, `drop_chance`, `observe_reward`, and `reward_ballot` as `pub(crate)` in `rust/crates/forge-data/src/recordings.rs` and updated `codex::recorded::snapshot` in `rust/crates/forge-data/src/codex/recorded.rs` to record `kill`/`slain` and `turnin` events and call the shared `recordings.rs` functions directly. Updated `docs/ARCHITECTURE.md` to reflect the aligned rules and added unit tests in `codex::recorded` ensuring that both pipelines produce matching drop chances and quest reward lists for identical sessions.
- **How It Was Verified**: Ran `forge data check-corrections` (0 error(s), 0 warning(s)), `forge guide check-generated` (clean), `cargo test --manifest-path rust/Cargo.toml --lib -p forge-data codex` (13 passed), `cargo test --manifest-path rust/Cargo.toml --lib -p forge-data recordings` (17 passed), `cargo test --manifest-path rust/Cargo.toml --test data -p forge-data -- codex` (3 passed), `cargo test --manifest-path rust/Cargo.toml --test addontest -p forge-addontest -- codex` (84 passed), and `node --test tests/codex-drops.test.js tests/codex-release.test.js` (10 passed).

<details>
<summary><b>Click to expand exact Code Diff (3 files changed, 292 insertions(+), 146 deletions(-))</b></summary>

```diff
diff --git a/docs/ARCHITECTURE.md b/docs/ARCHITECTURE.md
index ef5d3fd13..8713bf19f 100644
--- a/docs/ARCHITECTURE.md
+++ b/docs/ARCHITECTURE.md
@@ -5847,11 +5847,13 @@ was already built without it.
   days; the maintainer's own and vouched-for files always; `config.json`'s
   `minBuild`, `minVoters`), and only facts without who saw them: per NPC or
   object the items its loot windows held (NPC loot from a gathering skill
-  left out), with a chance (the median of each player's own share) only
-  when `minVoters` players with 10 loot windows each back it; per NPC its
-  merchant window's items (which also gives it the `vendor` and `sells:*`
-  roles); per quest the reward list most players' accept windows showed
-  (`trusted` when `minVoters` agree). Provenance `recordings`, ranked first.
+  left out), with a chance (the median of each player's own share of kills or
+  loot windows) only when `minVoters` players (or a vouched one) with 10 of
+  their own back it; per NPC its merchant window's items (which also gives it
+  the `vendor` and `sells:*` roles); per quest the reward list its quest
+  frames showed (a turn-in over an accept per player, decided by `Ballot`,
+  `trusted` when `minVoters` agree or a vouched player recorded it).
+  Provenance `recordings`, ranked first.
   A trusted reward list stands in for the web quest lists' for that quest;
   an untrusted one only fills a quest no other input gives rewards for.
 - Curated vendors (`curated_vendors`): in a guide that `collects` an item,
diff --git a/rust/crates/forge-data/src/codex/recorded.rs b/rust/crates/forge-data/src/codex/recorded.rs
index e12f4ac5c..0d02b56be 100644
--- a/rust/crates/forge-data/src/codex/recorded.rs
+++ b/rust/crates/forge-data/src/codex/recorded.rs
@@ -9,14 +9,17 @@
 //! thresholds apply), and every value is a fact without who recorded it:
 //!
 //! - `drops`/`objects`: per NPC or object, each item its loot windows held, with the number of
-//!   counted players who saw it and a `chance` (percent, the median of each player's own share of
-//!   loot windows) only on trusted evidence: `minVoters` players, or a vouched one, each with at
-//!   least [`MIN_WINDOWS`] loot windows of that NPC or object. NPC loot from a gathering skill
-//!   (skinning, herbs, ore) is left out.
+//!   counted players who saw it and a `chance` ([`crate::recordings::drop_chance`]: percent, the
+//!   median of each player's own share of kills or loot windows) only on trusted evidence:
+//!   `minVoters` players, or a vouched one, each with at least [`MIN_WINDOWS`] kills (or loot
+//!   windows, for an object) of that source. NPC loot from a gathering skill (skinning, herbs, ore)
+//!   is left out.
 //! - `vendors`: per NPC, the items its merchant window listed.
-//! - `rewards`: per quest, the reward items and choices its accept window showed, the list most
-//!   counted players saw (a tie leaves the quest out), with `voters` and `trusted` (`minVoters`
-//!   players agree on it, or a vouched one recorded it).
+//! - `rewards`: per quest, the reward items and choices its quest frames showed
+//!   ([`crate::recordings::observe_reward`] and [`crate::recordings::Ballot::decide`]: a turn-in
+//!   ranks over an accept per player, vouched evidence ranks first, and the first recorded breaks a
+//!   tie), with `voters` and `trusted` (`minVoters` players agree on it, or a vouched one recorded
+//!   it).
 //!
 //! The merge (`codex::merge`) adds a recorded drop, object, vendor or reward source like any other
 //! input's, under the provenance `recordings`; a quest's trusted reward list replaces the other
@@ -27,46 +30,37 @@ use std::collections::{BTreeMap, BTreeSet};
 use std::path::Path;
 
 use forge_guide::js::{parse_json, stringify, Obj, V};
-use indexmap::IndexSet;
+use indexmap::IndexMap;
 
-use super::util::{math_round, nums};
-use crate::recordings::{read_recordings, trusted, Recordings};
+use super::util::nums;
+use crate::recordings::{drop_chance, drop_samples, observe_reward, read_recordings, reward_ballot, Recordings, MIN_KILLS};
 
 /// The snapshot's file in data/codex.
 pub const FILE: &str = "recorded.json";
-/// Loot windows a player needs of an NPC or object before their share counts toward a chance.
-pub const MIN_WINDOWS: usize = 10;
+/// Kills (or loot windows, for an object) a player needs of a source before their share counts
+/// toward a trusted drop chance.
+pub const MIN_WINDOWS: usize = MIN_KILLS as usize;
 
 fn pos_int(v: &V) -> Option<i64> {
 	let n = v.as_num()?;
 	(n.is_finite() && n > 0.0 && n.fract() == 0.0).then_some(n as i64)
 }
 
-fn median(mut v: Vec<f64>) -> f64 {
-	v.sort_by(|a, b| a.partial_cmp(b).unwrap());
-	let n = v.len();
-	if n % 2 == 1 {
-		v[n / 2]
-	} else {
-		(v[n / 2 - 1] + v[n / 2]) / 2.0
-	}
-}
-
 #[derive(Default)]
 struct Loot {
 	/// voter -> loot windows of this source.
-	windows: BTreeMap<String, usize>,
+	windows: IndexMap<String, f64>,
 	/// item -> voter -> windows holding it.
-	items: BTreeMap<i64, BTreeMap<String, usize>>,
+	items: BTreeMap<i64, IndexMap<String, f64>>,
 }
 
 /// The snapshot of `recs` (every session already marked by `establish`).
 pub fn snapshot(recs: &Recordings) -> V {
 	let min_voters = recs.min_voters;
+	let mut kills: BTreeMap<i64, IndexMap<String, f64>> = BTreeMap::new();
 	let mut loot: [BTreeMap<i64, Loot>; 2] = [BTreeMap::new(), BTreeMap::new()];
 	let mut vendors: BTreeMap<i64, BTreeMap<i64, BTreeSet<String>>> = BTreeMap::new();
-	// quest -> voter -> (items, choices), the voter's latest accept window.
-	let mut rewards: BTreeMap<i64, BTreeMap<String, (Vec<i64>, Vec<i64>)>> = BTreeMap::new();
+	let mut rewards: BTreeMap<i64, IndexMap<String, (u8, V)>> = BTreeMap::new();
 	let (mut counted, mut through) = (0usize, 0f64);
 	let mut voters: BTreeSet<String> = BTreeSet::new();
 	for r in recs.iter() {
@@ -78,9 +72,20 @@ pub fn snapshot(recs: &Recordings) -> V {
 		}
 		counted += 1;
 		let who = r.voter();
+		let schema2 = r.rec.get("schema").as_num() == Some(2.0);
 		voters.insert(who.clone());
 		for ev in r.rec.get("events").items() {
 			match ev.get("e").as_str() {
+				Some("slain") => {
+					if let Some(npc) = pos_int(&ev.get("npc")).filter(|_| !ev.get("denied").truthy()) {
+						*kills.entry(npc).or_default().entry(who.clone()).or_insert(0.0) += 1.0;
+					}
+				}
+				Some("kill") if schema2 => {
+					if let Some(npc) = pos_int(&ev.get("npc")).filter(|_| !ev.get("denied").truthy()) {
+						*kills.entry(npc).or_default().entry(who.clone()).or_insert(0.0) += 1.0;
+					}
+				}
 				Some("drop") => {
 					let (k, id) = match (pos_int(&ev.get("npc")), pos_int(&ev.get("object"))) {
 						(Some(n), _) if !ev.get("gather").truthy() => (0, n),
@@ -89,10 +94,10 @@ pub fn snapshot(recs: &Recordings) -> V {
 						_ => continue,
 					};
 					let l = loot[k].entry(id).or_default();
-					*l.windows.entry(who.clone()).or_default() += 1;
+					*l.windows.entry(who.clone()).or_insert(0.0) += 1.0;
 					let seen: BTreeSet<i64> = ev.get("items").items().iter().filter_map(|i| pos_int(&i.get("item"))).collect();
 					for item in seen {
-						*l.items.entry(item).or_default().entry(who.clone()).or_default() += 1;
+						*l.items.entry(item).or_default().entry(who.clone()).or_insert(0.0) += 1.0;
 					}
 				}
 				Some("vendor") => {
@@ -104,14 +109,14 @@ pub fn snapshot(recs: &Recordings) -> V {
 					}
 				}
 				Some("accept") => {
-					let Some(q) = pos_int(&ev.get("q")) else { continue };
-					let ids = |k: &str| -> Vec<i64> {
-						let mut v: Vec<i64> = ev.get(k).items().iter().filter_map(pos_int).collect();
-						v.sort_unstable();
-						v.dedup();
-						v
-					};
-					rewards.entry(q).or_default().insert(who.clone(), (ids("items"), ids("choices")));
+					if let Some(q) = pos_int(&ev.get("q")) {
+						observe_reward(rewards.entry(q).or_default(), &ev, 1, &who);
+					}
+				}
+				Some("turnin") => {
+					if let Some(q) = pos_int(&ev.get("q")) {
+						observe_reward(rewards.entry(q).or_default(), &ev, 2, &who);
+					}
 				}
 				_ => {}
 			}
@@ -129,31 +134,17 @@ pub fn snapshot(recs: &Recordings) -> V {
 	for (k, name) in [(0, "drops"), (1, "objects")] {
 		let o = Obj::new();
 		for (id, l) in &loot[k] {
+			let of_by = drop_samples(k == 0, Some(&l.windows), kills.get(id));
 			let its = Obj::new();
 			for (item, by) in &l.items {
 				let e = Obj::new();
 				e.set("voters", by.len());
-				// Each player with enough loot windows of their own: their share.
-				let shares: Vec<(String, f64)> = by
-					.keys()
-					.filter_map(|w| {
-						let n = *l.windows.get(w)?;
-						(n >= MIN_WINDOWS).then(|| (w.clone(), by[w] as f64 / n as f64 * 100.0))
-					})
-					.collect();
-				let whose: IndexSet<String> = shares.iter().map(|(w, _)| w.clone()).collect();
-				// Players with enough windows who never saw the item count as 0%.
-				let mut all: Vec<f64> = shares.iter().map(|(_, s)| *s).collect();
-				let zero: Vec<String> = l.windows.iter().filter(|(w, n)| **n >= MIN_WINDOWS && !by.contains_key(*w)).map(|(w, _)| w.clone()).collect();
-				all.extend(zero.iter().map(|_| 0.0));
-				let mut every = whose.clone();
-				every.extend(zero);
-				let chance = if trusted(&every, min_voters) && !all.is_empty() { math_round(median(all) * 100.0) / 100.0 } else { 0.0 };
-				e.set("chance", if chance > 0.0 { V::Num(chance) } else { V::Null });
+				let chance = drop_chance(&of_by, by, min_voters).filter(|dc| dc.trusted).map(|dc| V::Num(dc.chance)).unwrap_or(V::Null);
+				e.set("chance", chance);
 				its.set(&item.to_string(), V::Obj(e));
 			}
 			let src = Obj::new();
-			src.set("windows", l.windows.values().sum::<usize>());
+			src.set("windows", l.windows.values().sum::<f64>() as usize);
 			src.set("items", V::Obj(its));
 			o.set(&id.to_string(), V::Obj(src));
 		}
@@ -166,25 +157,17 @@ pub fn snapshot(recs: &Recordings) -> V {
 	out.set("vendors", V::Obj(v));
 	let rw = Obj::new();
 	for (q, by) in &rewards {
-		// The list most players saw; a tie between lists leaves the quest out.
-		let mut ballot: BTreeMap<&(Vec<i64>, Vec<i64>), IndexSet<String>> = BTreeMap::new();
-		for (w, list) in by {
-			ballot.entry(list).or_default().insert(w.clone());
-		}
-		let best = ballot.values().map(IndexSet::len).max().unwrap_or(0);
-		let top: Vec<_> = ballot.iter().filter(|(_, ws)| ws.len() == best).collect();
-		if top.len() != 1 {
-			continue;
-		}
-		let ((items, choices), ws) = top[0];
-		if items.is_empty() && choices.is_empty() {
+		let ballot = reward_ballot(by);
+		let Some((r, tr)) = ballot.decide(false, min_voters) else { continue };
+		if r.get("items").items().is_empty() && r.get("choices").items().is_empty() {
 			continue;
 		}
+		let (_, ws) = ballot.best().unwrap();
 		let e = Obj::new();
-		e.set("items", nums(&items.iter().map(|i| *i as f64).collect::<Vec<f64>>()));
-		e.set("choices", nums(&choices.iter().map(|i| *i as f64).collect::<Vec<f64>>()));
+		e.set("items", r.get("items"));
+		e.set("choices", r.get("choices"));
 		e.set("voters", ws.len());
-		e.set("trusted", trusted(ws, min_voters));
+		e.set("trusted", tr);
 		rw.set(&q.to_string(), V::Obj(e));
 	}
 	out.set("rewards", V::Obj(rw));
@@ -235,13 +218,27 @@ pub fn source_meta(rec: &V) -> Option<V> {
 #[cfg(test)]
 mod tests {
 	use super::*;
-	use crate::recordings::Recording;
+	use crate::recordings::{apply_recordings, Db, Recording};
+	use crate::spawns::DEFAULTS;
 
 	fn session(who: &str, counts: bool, events: &str) -> Recording {
-		let rec = parse_json(&format!(r#"{{"schema":2,"contributor":"{who}","events":[{events}]}}"#)).unwrap();
+		let rec = parse_json(&format!(r#"{{"schema":2,"contributor":"{who}","locale":"enUS","events":[{events}]}}"#)).unwrap();
 		Recording { file: format!("{who}.json"), rec, counts }
 	}
 
+	fn kills_and_drops(npc: i64, kills: usize, drops: usize, item: i64) -> String {
+		(0..kills)
+			.flat_map(|i| {
+				let mut v = vec![format!(r#"{{"e":"kill","npc":{npc},"cls":"normal"}}"#)];
+				if i < drops {
+					v.push(format!(r#"{{"e":"drop","npc":{npc},"items":[{{"item":{item},"count":1}},{{"item":2589,"count":1}}]}}"#));
+				}
+				v
+			})
+			.collect::<Vec<_>>()
+			.join(",")
+	}
+
 	fn windows(npc: i64, n: usize, with: usize, item: i64) -> String {
 		(0..n)
 			.map(
@@ -260,13 +257,14 @@ mod tests {
 	#[test]
 	fn the_snapshot_keeps_counted_players_facts_and_gives_a_chance_only_on_trusted_evidence() {
 		let accept = |q: i64, items: &str, choices: &str| format!(r#"{{"e":"accept","q":{q},"items":[{items}],"choices":[{choices}]}}"#);
+		let turnin = |q: i64, items: &str, choices: &str| format!(r#"{{"e":"turnin","q":{q},"items":[{items}],"choices":[{choices}]}}"#);
 		let list = vec![
 			session(
 				"p1",
 				true,
 				&[
-					windows(100, 10, 2, 5),
-					windows(101, 3, 3, 7),
+					kills_and_drops(100, 10, 2, 5),
+					kills_and_drops(101, 3, 3, 7),
 					r#"{"e":"drop","npc":200,"gather":"skin","items":[{"item":2318,"count":1}]}"#.into(),
 					accept(7, "", "1,2"),
 					accept(8, "3", ""),
@@ -274,17 +272,17 @@ mod tests {
 				]
 				.join(","),
 			),
-			session("p2", true, &[windows(100, 10, 3, 5), accept(7, "", "2,1"), accept(8, "4", "")].join(",")),
+			session("p2", true, &[kills_and_drops(100, 10, 3, 5), accept(7, "", "9"), turnin(7, "", "1,2"), accept(8, "4", "")].join(",")),
 			session("p3", true, &[windows(100, 20, 8, 5), accept(7, "", "1,2"), r#"{"e":"drop","object":3000,"items":[{"item":2770,"count":2}]}"#.into()].join(",")),
 			session("p4", false, &[r#"{"e":"vendor","npc":945,"items":[{"item":2131,"price":54,"stack":1,"qty":-1}]}"#.to_string(), accept(10, "6", "")].join(",")),
 			session("p1", true, r#"{"e":"vendor","npc":945,"items":[{"item":2130,"price":54,"stack":1,"qty":-1}]}"#),
 		];
 		let snap = snapshot(&Recordings { list, min_voters: 3 });
 		let s = |v: &V| stringify(v, None);
-		// Three players with ten windows each: 20%, 30%, 40%.
+		// Three players with ten kills/windows each: 20%, 30%, 40%.
 		assert_eq!(s(&snap.get("drops").get("100").get("items").get("5")), r#"{"voters":3,"chance":30}"#);
-		assert_eq!(s(&snap.get("drops").get("100").get("windows")), "40");
-		// One player with three windows: the row, no chance.
+		assert_eq!(s(&snap.get("drops").get("100").get("windows")), "25");
+		// One player with three kills: the row, no chance.
 		assert_eq!(s(&snap.get("drops").get("101").get("items").get("7")), r#"{"voters":1,"chance":null}"#);
 		assert!(snap.get("drops").get("200").is_undef(), "skinning is no drop");
 		assert_eq!(s(&snap.get("objects").get("3000").get("items")), r#"{"2770":{"voters":1,"chance":null}}"#);
@@ -292,8 +290,124 @@ mod tests {
 		assert_eq!(s(&snap.get("vendors")), r#"{"945":[2130]}"#);
 		assert!(snap.get("rewards").get("10").is_undef());
 		assert_eq!(s(&snap.get("rewards").get("7")), r#"{"items":[],"choices":[1,2],"voters":3,"trusted":true}"#);
-		assert!(snap.get("rewards").get("8").is_undef(), "two players, two lists: a tie");
+		assert_eq!(s(&snap.get("rewards").get("8")), r#"{"items":[3],"choices":[],"voters":1,"trusted":false}"#, "a tie keeps the first recorded list");
 		assert_eq!(s(&snap.get("rewards").get("9")), r#"{"items":[5],"choices":[],"voters":1,"trusted":false}"#);
 		assert_eq!(s(&snap.get("meta")), r#"{"sessions":5,"counted":4,"voters":3,"minVoters":3,"minWindows":10,"through":null}"#);
 	}
+
+	#[test]
+	fn the_snapshot_and_the_data_build_give_the_same_drop_chances_and_quest_rewards_for_the_same_sessions() {
+		let accept = |q: i64, items: &str, choices: &str| format!(r#"{{"e":"accept","q":{q},"items":[{items}],"choices":[{choices}]}}"#);
+		let turnin = |q: i64, items: &str, choices: &str| format!(r#"{{"e":"turnin","q":{q},"items":[{items}],"choices":[{choices}]}}"#);
+		let obj_drops = |obj: i64, n: usize, with: usize, item: i64| {
+			(0..n)
+				.map(|i| {
+					if i < with {
+						format!(r#"{{"e":"drop","object":{obj},"items":[{{"item":{item},"count":1}}]}}"#)
+					} else {
+						format!(r#"{{"e":"drop","object":{obj},"items":[]}}"#)
+					}
+				})
+				.collect::<Vec<_>>()
+				.join(",")
+		};
+		let list = vec![
+			session(
+				"aaaaaaaaaaaaaaaa",
+				true,
+				&[
+					// 20 kills (plus one denied kill that does not count), 2 drops -> 10%.
+					kills_and_drops(1501, 20, 2, 2933),
+					r#"{"e":"kill","npc":1501,"cls":"normal","denied":true}"#.into(),
+					// 10 kills of 1502 with no drops -> 0% share.
+					kills_and_drops(1502, 10, 0, 2933),
+					obj_drops(3000, 10, 1, 2770),
+					// Quest 7: turn-in (weight 2) beats accept (weight 1), and a later accept does not overwrite it.
+					accept(7, "", "99"),
+					turnin(7, "10", "1,2"),
+					accept(7, "", "88"),
+					// Quest 8: two unvouched lists tie; first recorded wins (untrusted).
+					accept(8, "3", "30"),
+					// Quest 9: two unvouched players vs one vouched player; vouched wins (trusted).
+					accept(9, "5", ""),
+				]
+				.join(","),
+			),
+			session(
+				"bbbbbbbbbbbbbbbb",
+				true,
+				&[
+					// 20 kills, 5 drops -> 25%.
+					kills_and_drops(1501, 20, 5, 2933),
+					kills_and_drops(1502, 10, 2, 2933),
+					obj_drops(3000, 10, 3, 2770),
+					// A schema-2 turn-in without frame items/choices/rewardMoney keeps the accept's rewards.
+					accept(7, "10", "1,2"),
+					r#"{"e":"turnin","q":7,"xpReward":450,"money":100}"#.into(),
+					accept(8, "4", "40"),
+					accept(9, "5", ""),
+				]
+				.join(","),
+			),
+			session(
+				"cccccccccccccccc",
+				true,
+				&[
+					// 400 kills, 400 drops -> 100% outlier; median of [10%, 25%, 100%] is 25%.
+					kills_and_drops(1501, 400, 400, 2933),
+					kills_and_drops(1502, 10, 4, 2933),
+					obj_drops(3000, 20, 10, 2770),
+					turnin(7, "10", "1,2"),
+				]
+				.join(","),
+			),
+			session("sent-alice", true, &accept(9, "6", "60")),
+		];
+		let recs = Recordings { list, min_voters: 3 };
+		let snap = snapshot(&recs);
+
+		let obj = |j: &str| match parse_json(j).unwrap() {
+			V::Obj(o) => o,
+			_ => unreachable!(),
+		};
+		let quests = obj(r#"{
+			"7":{"id":7,"title":"Q7","level":5,"start":{"npc":[],"object":[],"item":[]},"end":{"npc":[],"object":[]},"objectives":[],"rewards":{"items":[1],"choices":[],"money":null},"source":["pfquest"]},
+			"8":{"id":8,"title":"Q8","level":5,"start":{"npc":[],"object":[],"item":[]},"end":{"npc":[],"object":[]},"objectives":[],"source":["pfquest"]},
+			"9":{"id":9,"title":"Q9","level":5,"start":{"npc":[],"object":[],"item":[]},"end":{"npc":[],"object":[]},"objectives":[],"rewards":{"items":[1],"choices":[],"money":null},"source":["pfquest"]}
+		}"#);
+		let npcs = obj(r#"{"1501":{"id":1501,"name":"A","spawns":{}},"1502":{"id":1502,"name":"B","spawns":{}}}"#);
+		let objects = obj(r#"{"3000":{"id":3000,"name":"Chest","spawns":{}}}"#);
+		let items = obj(r#"{
+			"2933":{"id":2933,"name":"Seal","drops":{"npc":{"1501":1,"1502":1},"object":{}},"vendors":[]},
+			"2589":{"id":2589,"name":"Linen","drops":{"npc":{"1501":1,"1502":1},"object":{}},"vendors":[]},
+			"2770":{"id":2770,"name":"Copper","drops":{"npc":{},"object":{"3000":1}},"vendors":[]}
+		}"#);
+		let (zones, factions, instances) = (obj("{}"), obj("{}"), obj("{}"));
+		let db = Db { quests: &quests, npcs: &npcs, objects: &objects, items: &items, zones: &zones, factions: &factions, instances: &instances };
+		apply_recordings(&db, &recs.list, recs.min_voters, DEFAULTS, &|_| false, false);
+
+		let s = |v: &V| stringify(v, None);
+		for (npc, expected) in [("1501", 25.0), ("1502", 20.0)] {
+			let data_chance = items.get("2933").get("drops").get("npc").get(npc).as_num();
+			let codex_chance = snap.get("drops").get(npc).get("items").get("2933").get("chance").as_num();
+			assert_eq!(data_chance, Some(expected), "npc {npc}");
+			assert_eq!(codex_chance, data_chance, "npc {npc}");
+		}
+		let data_obj = items.get("2770").get("drops").get("object").get("3000").as_num();
+		let codex_obj = snap.get("objects").get("3000").get("items").get("2770").get("chance").as_num();
+		assert_eq!(data_obj, Some(30.0));
+		assert_eq!(codex_obj, data_obj);
+
+		for (q, expected_items, expected_choices, expected_trusted) in [("7", "[10]", "[1,2]", true), ("8", "[3]", "[30]", false), ("9", "[6]", "[60]", true)] {
+			let dr = quests.get(q).get("rewards");
+			let cr = snap.get("rewards").get(q);
+			assert_eq!(s(&dr.get("items")), expected_items, "quest {q} items");
+			assert_eq!(s(&dr.get("choices")), expected_choices, "quest {q} choices");
+			assert_eq!(s(&cr.get("items")), s(&dr.get("items")), "quest {q} items match");
+			assert_eq!(s(&cr.get("choices")), s(&dr.get("choices")), "quest {q} choices match");
+			assert_eq!(cr.get("trusted").truthy(), expected_trusted, "quest {q} trusted");
+			let tag = if expected_trusted { "recorder:rewards" } else { "recorder-gap:rewards" };
+			assert!(quests.get(q).get("source").items().iter().any(|x| x.as_str() == Some(tag)), "quest {q} source tag {tag}");
+		}
+	}
 }
diff --git a/rust/crates/forge-data/src/recordings.rs b/rust/crates/forge-data/src/recordings.rs
index ab23e54b2..9e987ab40 100644
--- a/rust/crates/forge-data/src/recordings.rs
+++ b/rust/crates/forge-data/src/recordings.rs
@@ -361,7 +361,7 @@ const OBJECT_KEYS_SIM: [&str; 5] = ["id", "name", "roles", "spawns", "instances"
 const QUEST_KEYS: [&str; 20] =
 	["id", "title", "level", "minLevel", "races", "classes", "start", "end", "pre", "excl", "next", "objectives", "objectiveText", "zone", "xp", "xpSamples", "rewards", "skill", "event", "source"];
 const FACTION_KEYS: [&str; 5] = ["id", "name", "side", "parent", "source"];
-const MIN_KILLS: f64 = 10.0;
+pub(crate) const MIN_KILLS: f64 = 10.0;
 const ENTRANCE_RADIUS: f64 = 1.5;
 const FULL_XP_LEVELS: f64 = 5.0;
 
@@ -415,7 +415,7 @@ type Backers<K> = IndexMap<K, IndexSet<String>>;
 
 /// The players behind each value recorded for one field, in the order first recorded.
 #[derive(Default)]
-struct Ballot(IndexMap<String, (V, IndexSet<String>)>);
+pub(crate) struct Ballot(IndexMap<String, (V, IndexSet<String>)>);
 
 /// How strongly a set of players backs a value: vouched evidence first, then the number of players.
 fn weight(w: &IndexSet<String>) -> (bool, usize) {
@@ -423,13 +423,13 @@ fn weight(w: &IndexSet<String>) -> (bool, usize) {
 }
 
 impl Ballot {
-	fn vote(&mut self, v: V, who: &str) {
+	pub(crate) fn vote(&mut self, v: V, who: &str) {
 		let e = self.0.entry(stringify(&v, None)).or_insert_with(|| (v, IndexSet::new()));
 		e.1.insert(who.to_string());
 	}
 
 	/// The value the most players back (ties: the first recorded), with its players.
-	fn best(&self) -> Option<(&V, &IndexSet<String>)> {
+	pub(crate) fn best(&self) -> Option<(&V, &IndexSet<String>)> {
 		let mut best: Option<(&V, &IndexSet<String>)> = None;
 		for (v, w) in self.0.values() {
 			if best.is_none_or(|(_, b)| weight(w) > weight(b)) {
@@ -442,7 +442,7 @@ impl Ballot {
 	/// The value to write over `cur` (`present`: another source has one) and whether it is
 	/// trusted: the best value fills a gap, and replaces or confirms a present one only on
 	/// trusted evidence.
-	fn decide(&self, present: bool, min: usize) -> Option<(V, bool)> {
+	pub(crate) fn decide(&self, present: bool, min: usize) -> Option<(V, bool)> {
 		let (v, who) = self.best()?;
 		let t = trusted(who, min);
 		(!present || t).then(|| (v.clone(), t))
@@ -596,11 +596,11 @@ struct Overlay<'a> {
 	factions_seen: IndexMap<i64, IndexSet<String>>,
 	faction_parent: IndexMap<i64, Ballot>,
 	faction_side: IndexMap<i64, Ballot>,
-	// finds, counted per contributor index (one player's numbers never pool with another's)
-	kills: FxHashMap<i64, IndexMap<usize, f64>>,
-	windows: [FxHashMap<i64, IndexMap<usize, f64>>; 2],
-	/// source -> item -> contributor -> loot windows holding the item.
-	loots: [IndexMap<i64, IndexMap<i64, IndexMap<usize, f64>>>; 2],
+	// finds, counted per player (one player's numbers never pool with another's)
+	kills: FxHashMap<i64, IndexMap<String, f64>>,
+	windows: [FxHashMap<i64, IndexMap<String, f64>>; 2],
+	/// source -> item -> player -> loot windows holding the item.
+	loots: [IndexMap<i64, IndexMap<i64, IndexMap<String, f64>>>; 2],
 	found: [IndexMap<i64, ByMap>; 2],
 	killed: IndexMap<i64, ByMap>,
 	rank: IndexMap<i64, Ballot>,
@@ -1090,14 +1090,14 @@ impl<'a> Overlay<'a> {
 					return true;
 				}
 				let k = k_ix(kind);
-				let ci = self.ci;
-				*self.windows[k].entry(id).or_default().entry(ci).or_insert(0.0) += 1.0;
+				let who = self.voter.clone();
+				*self.windows[k].entry(id).or_default().entry(who.clone()).or_insert(0.0) += 1.0;
 				let mut seen: IndexSet<i64> = IndexSet::new();
 				for i in &list {
 					seen.insert(pos_int(&i.get("item")).unwrap());
 				}
 				for item in seen {
-					*self.loots[k].entry(id).or_default().entry(item).or_default().entry(ci).or_insert(0.0) += 1.0;
+					*self.loots[k].entry(id).or_default().entry(item).or_default().entry(who.clone()).or_insert(0.0) += 1.0;
 					self.item_entry(item, &V::Undef);
 				}
 				self.loots[k].entry(id).or_default();
@@ -1158,7 +1158,7 @@ impl<'a> Overlay<'a> {
 			push_point(&mut self.killed, npc, &c);
 		}
 		if !ev.get("denied").truthy() {
-			*self.kills.entry(npc).or_default().entry(self.ci).or_insert(0.0) += 1.0;
+			*self.kills.entry(npc).or_default().entry(self.voter.clone()).or_insert(0.0) += 1.0;
 		}
 	}
 
@@ -1231,33 +1231,13 @@ impl<'a> Overlay<'a> {
 		for kind in ["npc", "object"] {
 			let k = k_ix(kind);
 			for (&id, m) in &self.loots[k] {
-				// Each player's kills of the source (or loot windows, the more of the two), or its
-				// loot windows of an object.
-				let mut of_by: IndexMap<usize, f64> = self.windows[k].get(&id).cloned().unwrap_or_default();
-				if kind == "npc" {
-					for (&c, &n) in self.kills.get(&id).into_iter().flatten() {
-						let of = of_by.entry(c).or_insert(0.0);
-						*of = of.max(n);
-					}
-				}
+				let of_by = drop_samples(kind == "npc", self.windows[k].get(&id), self.kills.get(&id));
 				for (&item, n_by) in m {
 					let it = db.items.get(&item.to_string());
 					if !it.truthy() {
 						continue;
 					}
-					// From MIN_KILLS of their own, `min` players (or a vouched one) make a trusted
-					// chance that replaces another source's; otherwise the chance only fills a
-					// gap. Either way it is the median of each player's chance.
-					let all: Vec<usize> = of_by.iter().filter(|(_, of)| **of > 0.0).map(|(c, _)| *c).collect();
-					let strong: Vec<usize> = all.iter().copied().filter(|c| of_by[c] >= MIN_KILLS).collect();
-					let backers: IndexSet<String> = strong.iter().map(|c| self.contributors[*c].clone()).collect();
-					let trust = trusted(&backers, self.min);
-					let group = if trust { strong } else { all };
-					let n_of = |c: &usize| n_by.get(c).copied().unwrap_or(0.0);
-					let chance = median(group.iter().map(|c| 100.0 * n_of(c) / of_by[c]).collect());
-					if chance.is_nan() || chance <= 0.0 {
-						continue;
-					}
+					let Some(dc) = drop_chance(&of_by, n_by, self.min) else { continue };
 					if !it.get("drops").truthy() {
 						let d = Obj::new();
 						d.set("npc", Obj::new());
@@ -1268,10 +1248,10 @@ impl<'a> Overlay<'a> {
 						it.get("drops").set(kind, Obj::new());
 					}
 					let cur = it.get("drops").get(kind).get(&id.to_string());
-					if !cur.is_nullish() && !trust {
+					if !cur.is_nullish() && !dc.trusted {
 						continue;
 					}
-					it.get("drops").get(kind).set(&id.to_string(), round2(chance));
+					it.get("drops").get(kind).set(&id.to_string(), dc.chance);
 					chances += 1;
 					if !it.get("seen").truthy() {
 						it.set("seen", Obj::new());
@@ -1281,10 +1261,10 @@ impl<'a> Overlay<'a> {
 					}
 					// The loot and kills behind the chance.
 					let s = Obj::new();
-					s.set("n", group.iter().map(n_of).sum::<f64>());
-					s.set("of", group.iter().map(|c| of_by[c]).sum::<f64>());
+					s.set("n", dc.n);
+					s.set("of", dc.of);
 					it.get("seen").get(kind).set(&id.to_string(), s);
-					tag_field(&it, "drops", trust);
+					tag_field(&it, "drops", dc.trusted);
 				}
 			}
 		}
@@ -1486,40 +1466,90 @@ impl<'a> Overlay<'a> {
 				a.0.borrow_mut().insert(0, V::Num(id as f64));
 			}
 		}
-		let mut rewards = Ballot::default();
-		for (who, (_, r)) in &v.rewards {
-			rewards.vote(r.clone(), who);
-		}
-		if let Some((r, tr)) = rewards.decide(!q.get("rewards").is_nullish(), self.min) {
+		if let Some((r, tr)) = reward_ballot(&v.rewards).decide(!q.get("rewards").is_nullish(), self.min) {
 			q.set("rewards", r);
 			tag_field(q, "rewards", tr);
 		}
 	}
 }
 
+pub(crate) struct DropChance {
+	pub chance: f64,
+	pub trusted: bool,
+	pub n: f64,
+	pub of: f64,
+}
+
+/// Each player's kills of the source (or loot windows, the more of the two), or its loot windows
+/// of an object.
+pub(crate) fn drop_samples(npc: bool, windows: Option<&IndexMap<String, f64>>, kills: Option<&IndexMap<String, f64>>) -> IndexMap<String, f64> {
+	let mut of_by = windows.cloned().unwrap_or_default();
+	if npc {
+		for (who, &n) in kills.into_iter().flatten() {
+			let of = of_by.entry(who.clone()).or_insert(0.0);
+			*of = of.max(n);
+		}
+	}
+	of_by
+}
+
+/// From [`MIN_KILLS`] of their own, `min` players (or a vouched one) make a trusted chance that
+/// replaces another source's; otherwise the chance only fills a gap. Either way it is the median
+/// of each player's chance.
+pub(crate) fn drop_chance(of_by: &IndexMap<String, f64>, n_by: &IndexMap<String, f64>, min: usize) -> Option<DropChance> {
+	let all: Vec<&String> = of_by.iter().filter(|(_, of)| **of > 0.0).map(|(c, _)| c).collect();
+	let strong: Vec<&String> = all.iter().copied().filter(|c| of_by[*c] >= MIN_KILLS).collect();
+	let backers: IndexSet<String> = strong.iter().map(|c| (*c).clone()).collect();
+	let trust = trusted(&backers, min);
+	let group = if trust { strong } else { all };
+	let n_of = |c: &&String| n_by.get(*c).copied().unwrap_or(0.0);
+	let chance = median(group.iter().map(|c| 100.0 * n_of(c) / of_by[*c]).collect());
+	if chance.is_nan() || chance <= 0.0 {
+		return None;
+	}
+	Some(DropChance {
+		chance: round2(chance),
+		trusted: trust,
+		n: group.iter().map(n_of).sum(),
+		of: group.iter().map(|c| of_by[*c]).sum(),
+	})
+}
+
+/// The rewards a quest frame showed (items, choices, money), the player's latest of the highest
+/// weight (a turn-in 2 over an accept 1).
+pub(crate) fn observe_reward(rewards: &mut IndexMap<String, (u8, V)>, ev: &V, weight: u8, who: &str) {
+	let items = ids(&ev.get("items"));
+	let choices = ids(&ev.get("choices"));
+	let frame_money = pos_int(&ev.get("rewardMoney"));
+	if items.is_empty() && choices.is_empty() && frame_money.is_none() {
+		return;
+	}
+	let money = frame_money.or(if weight == 2 { pos_int(&ev.get("money")) } else { None });
+	if rewards.get(who).is_some_and(|(w, _)| *w > weight) {
+		return;
+	}
+	let o = Obj::new();
+	o.set("items", nums(&items.iter().map(|&x| x as f64).collect::<Vec<_>>()));
+	o.set("choices", nums(&choices.iter().map(|&x| x as f64).collect::<Vec<_>>()));
+	o.set("money", money.map(|m| V::Num(m as f64)).unwrap_or(V::Null));
+	rewards.insert(who.to_string(), (weight, V::Obj(o)));
+}
+
+pub(crate) fn reward_ballot(rewards: &IndexMap<String, (u8, V)>) -> Ballot {
+	let mut rewards_ballot = Ballot::default();
+	for (who, (_, r)) in rewards {
+		rewards_ballot.vote(r.clone(), who);
+	}
+	rewards_ballot
+}
+
 impl QuestVotes {
 	fn giver(&mut self, side: &'static str, kind: &'static str, id: i64, who: &str) {
 		self.givers.entry((side, kind)).or_default().entry(id).or_default().insert(who.to_string());
 	}
 
-	/// The rewards a quest frame showed (items, choices, money), the player's latest of the
-	/// highest weight.
 	fn reward(&mut self, ev: &V, weight: u8, who: &str) {
-		let items = ids(&ev.get("items"));
-		let choices = ids(&ev.get("choices"));
-		let frame_money = pos_int(&ev.get("rewardMoney"));
-		if items.is_empty() && choices.is_empty() && frame_money.is_none() {
-			return;
-		}
-		let money = frame_money.or(if weight == 2 { pos_int(&ev.get("money")) } else { None });
-		if self.rewards.get(who).is_some_and(|(w, _)| *w > weight) {
-			return;
-		}
-		let o = Obj::new();
-		o.set("items", nums(&items.iter().map(|&x| x as f64).collect::<Vec<_>>()));
-		o.set("choices", nums(&choices.iter().map(|&x| x as f64).collect::<Vec<_>>()));
-		o.set("money", money.map(|m| V::Num(m as f64)).unwrap_or(V::Null));
-		self.rewards.insert(who.to_string(), (weight, V::Obj(o)));
+		observe_reward(&mut self.rewards, ev, weight, who);
 	}
 }
 
```
</details>

---

## Open Issue Triage Summary (Skipped / Already Fixed / Covered by Open PRs)

All remaining candidate open issues were triaged against `forever-db/data/*.jsonl`, `forever-db/corrections/*.json`, `guides/`, `addon/`, `rust/`, and open PRs on `fitzforge/polestar`:

| Issue | Title | Verdict | Evidence / Reference |
|---|---|---|---|
| [#1150](https://github.com/fitzforge/polestar/issues/1150) | Turn in Valuable Vantages - tells me to hand in before completing | **Covered by `fix/issue-1051`** | Duplicate of [#1051](https://github.com/fitzforge/polestar/issues/1051) (`quest 98095` at Deathstalker Masoj `npc 270438` in step 14 of `06-hillsbrad-foothills.guide`). Linked in `fix/issue-1051` (`closes #1051, closes #1150`). |
| [#1124](https://github.com/fitzforge/polestar/issues/1124) | Return the Statuette not available from Karl Boran | **Covered by Open PR [#1132](https://github.com/fitzforge/polestar/pull/1132)** | Duplicate of [#1113](https://github.com/fitzforge/polestar/issues/1113) / [#1143](https://github.com/fitzforge/polestar/issues/1143); fixed on `upstream/fix/offer-audit` (`7a4ba95e9`, `0ea2d8706`) and `upstream/triage/batch-1143` (PR [#1153](https://github.com/fitzforge/polestar/pull/1153)). |
| [#1099](https://github.com/fitzforge/polestar/issues/1099) | Bug: 낚시하기전에 상점들러서 낙시대사야하고, 낙시도 배워야함 | **Reporter Mistaken (`resolution:no-action`)** | Quest `99127` (*A Net Disaster*) in `01-elwynn-forest.guide` step 25 is an `object-click` quest (`forever-db/data/mechanic.jsonl`) that loots `Half-Eaten Fish` (`item 286205`) from `Fishing Trap` containers (`object 694772`); no Fishing skill or pole is required. |
| [#1089](https://github.com/fitzforge/polestar/issues/1089) | 灰谷之旅无法领取，没看到这个任务 | **Needs Info / World State (`needs-info`)** | Quest `990` (*Trek to Ashenvale*, `npc 3694` Sentinel Selarin at `39.28,43.46`) follows `turnin 994` in `03-darkshore.guide` step 53; `forever-db` and player recordings (`fix/offer-audit`) confirm eligibility after `994`, and reporter's log shows `Own.notListed[990]` was never triggered. |
| [#1084](https://github.com/fitzforge/polestar/issues/1084) | Wrong step: Durotar, step 47 | **Already Fixed (`v0.3.2`)** | Reported on `0.3.1 Beta`; fixed in commit `9daf18dc1` (#807), which added `pre: [97281]` to `97282` (*Stormy Potential*) and moved `accept 97282 optional` after `turnin 97281 optional`. |
| [#1073](https://github.com/fitzforge/polestar/issues/1073) | Stockade Quests | **Already Fixed on `next`** | Duplicate of #1076; fixed in commit `14ddd1a94` (PR #1122), which preserves level-gated `preAny` prerequisites (`quest 303` *The Dark Iron War*, `minLevel 25`) in `Lookup.QuestChain` and `Dungeons.PickupPlan`. |
| [#1070](https://github.com/fitzforge/polestar/issues/1070) | Wrong step: The Barrens, step 71 | **Reporter Mistaken (`resolution:no-action`)** | Hezrul Bloodmark (`npc 3396`, quest `852`) spawns at `46.04, 41.14` in `forever-db/data/spawn.jsonl` (matching `04-the-barrens.guide`) and patrols Lushwater Oasis; reporter met him on the north shore while looking ahead from step 69. |
| [#1065](https://github.com/fitzforge/polestar/issues/1065) | Potential Sequence Bug | **Works as Designed (`resolution:no-action`)** | Guide header `levels 16-19` in `03-silverpine-forest.guide` is the character level range for the zone visit, not the quest level of `435` (*Escorting Erland*, level 11, still green at 18). |
| [#1053](https://github.com/fitzforge/polestar/issues/1053) | Rahauro in Thunder Bluff | **Already Fixed on `next`** | Fixed in commit `b651ed4d1` (#1032), which attaches `goal.z` (`LK().SpawnZ`) to dungeon pick-up goals so Rahauro (`npc 11833` on Elder Rise, `z = 159.1`) routes to his bluff floor instead of Mulgore below. |
| [#1043](https://github.com/fitzforge/polestar/issues/1043) | Arrow still needs to be alot more flexible. | **Already Fixed on `next`** | Fixed in commit `f97734300` (PR #1107, `changes/arrow-beside-trail.md`), adding 100-yard line-of-sight lookahead on baked legs. |
| [#1042](https://github.com/fitzforge/polestar/issues/1042) | Quest not availlable "Quelles sales mines !" in Westfall | **Already Fixed (`v0.3.4`)** | Reported on `0.3.2 Beta`; fixed in commit `a38c62a3c` (#1033), adding `preAll: [92742, 92744]` (*Testing the Wells* and *Murloc Gills*) to quest `92745` (*The State of the Mines*). |
| [#1041](https://github.com/fitzforge/polestar/issues/1041) | Order priority | **Already Fixed on `next`** | Fixed in commit `8f22b3179` (#1046 / PR #1050), setting `along: false` on quest `376` (*The Damned*) objectives 1 & 2 in `data/guide-fixes.json` so paws and wings stay in the Deathknell field step. |
| [#1040](https://github.com/fitzforge/polestar/issues/1040) | Quillboar quest ordering | **Already Fixed (`v0.3.3`)** | Reported on `0.3.2 Beta`; duplicate of #957, fixed in commit `dadd58919` (PR #962), ordering `accept 878` (*Tribes at War*) before `task 899` (*Consumed by Hatred*). |
| [#1035](https://github.com/fitzforge/polestar/issues/1035) | A Void Path | **Already Fixed (`v0.3.4`)** | Reported on `0.3.3 Beta`; duplicate of #1002 / #984, fixed in commit `aa75f61db` (PR #1009), adding `pre: [98251]` to `98252` (*A Void Path*) and moving `accept 98252` after `turnin 98251`. |
| [#1023](https://github.com/fitzforge/polestar/issues/1023) | Says to walk to mulgore while in TB when the quest pickup is in TB. | **Reporter Mistaken (`resolution:no-action`)** | Reporter's snapshot shows Step 9 (`[x] Accept Leaders of the Fang · quest 914` at Nara Wildmane in Thunder Bluff) was already completed and the guide was at Step 10 (`Step 9 of 13: Nara Wildmane (the guide is at step 10)`), routing down the elevator through Mulgore to The Barrens. |
| [#976](https://github.com/fitzforge/polestar/issues/976) | Fishing guide starter route optimization | **Enhancement / Feature Wish** | Architectural feature request for location-adaptive profession routes (tracked under broader profession guide design #681); quest `1141` is already included in `03-darkshore.guide`. |
