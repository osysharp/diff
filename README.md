# Osysharp.Diff

**Show what changed.** Hand it the text `git diff` prints — one file or many — and it draws it the way a reader expects:
each file under its path, old and new line numbers, removed and added lines on their tints with the changed words
marked inside them, unchanged runs folded to a count (opened in place when you give it the whole old and new text), in
one column or side by side.

Plain Osy#: no capability, no network, no JavaScript bundle. Every line is drawn as plain text.

## Use it

```osy
app Review {
  use Osysharp.Diff@1;
  use Osysharp.Ui;
  model "model/**/*.osy";
}
```

```osy
using Osysharp.Diff;

DiffView(change.Patch);                                              // unified, folds as counts
DiffView(change.Patch, oldText: before, newText: after);             // every fold opens in place
DiffView(change.Patch, layout: DiffLayout.Split);                    // old left, new right
DiffView(change.Patch, maxLines: 5000, label: "Changes to " + path); // draw more; name it for a test's within:

var files = ParseDiff(change.Patch);                                 // or read it as data
```

| Parameter | Default | What it does |
|---|---|---|
| `patch` | — | the unified diff text: `git diff`, `git show`, `diff -u` |
| `layout` | `DiffLayout.Unified` | `DiffLayout.Split` for side by side |
| `oldText` / `newText` | `""` | the whole old / new file (either is enough): every fold becomes a "Show N unchanged lines" button. One-file diffs only |
| `maxLines` | `1000` | the most lines of code drawn; the rest is "N more lines not shown" |
| `wordLimit` | `500` | the longest line (characters) whose changed words are marked |
| `label` | `"Diff"` | the accessible name, and what a test's `within:` scopes to |

## What it promises

- **Plain text, always.** Paths, hunk sections and lines are shown as the characters they are: `<script>`, `**bold**`,
  `{name}` and `[[link]]` included.
- **Bounded work.** One pass to parse; a hunk's head is only counted down, never used to size anything. Changed words
  are compared for lines of at most `wordLimit` characters and 200 words, and at most a million comparison cells per
  diff — a pair past that is drawn whole, with a note saying how many lines were not compared, and why.
- **A cap on what is drawn**, with the rest counted, files included.
- **Its theme is yours.** Every colour, the face and the sizes are `Diff*` tokens defaulting to your theme's surface, text and
  status ink; shadow any of them (`Colors { DiffAddedBg = Colors.SuccessTint; }`).

`osy docs ui-diff-kit` has the whole reference, including the token table.

## Tests

`kits/diff/tests` — the parser (several files, rename, no newline, binary, header look-alikes), the folds and opening
one, the word highlights and their caps, the line cap, the split pairing, and a gallery page read in the UI: hostile
text shown as text, a fold opened by a click. `osy test kits/diff/tests --pixels` photographs it in light and dark.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Diff;

[Test]
void a_git_diff_of_several_files_is_read_file_by_file() {
  var files = ParseDiff(MultiFilePatch());
  Assert.Equal(4, files.Count);

  Assert.Equal("src/app.txt", DiffPathOfFile(files[0]));
  Assert.Equal(DiffChange.Modified, files[0].Change);
  Assert.Equal(1, files[0].Hunks.Count);
  Assert.Equal("section one", files[0].Hunks[0].Section);
  Assert.Equal(1, files[0].Added);
  Assert.Equal(1, files[0].Removed);
  var lines = files[0].Hunks[0].Lines;
  Assert.Equal(4, lines.Count);
  Assert.Equal(DiffLineKind.Context, lines[0].Kind);
  Assert.Equal(1, lines[0].Old);
  Assert.Equal(1, lines[0].New);
  Assert.Equal(DiffLineKind.Removed, lines[1].Kind);
  Assert.Equal(2, lines[1].Old);
  Assert.Equal(0, lines[1].New);
  Assert.Equal("let total = price + tax;", lines[1].Text);
  Assert.Equal(DiffLineKind.Added, lines[2].Kind);
  Assert.Equal(2, lines[2].New);
  Assert.Equal(3, lines[3].Old);
  Assert.Equal(3, lines[3].New);

  Assert.Equal(DiffChange.Deleted, files[3].Change);
  Assert.Equal("gone.txt", DiffPathOfFile(files[3]));
  Assert.Equal(1, files[3].Removed);
}
```

From [`tests/diff.test.osy`](tests/diff.test.osy).

```osy
using Osysharp.Diff;

[Test]
void a_rename_keeps_both_paths_and_a_missing_final_newline_is_on_its_line() {
  var f = ParseDiff(MultiFilePatch())[1];
  Assert.Equal(DiffChange.Renamed, f.Change);
  Assert.Equal("old/name.txt", f.OldPath);
  Assert.Equal("new/name.txt", f.NewPath);
  Assert.Equal("renamed from old/name.txt", DiffChangeText(f));
  var lines = f.Hunks[0].Lines;
  Assert.Equal(3, lines.Count);
  Assert.False(lines[0].NoNewline);
  Assert.Equal("last", lines[1].Text);
  Assert.True(lines[1].NoNewline);
  Assert.Equal("last line", lines[2].Text);
  Assert.True(lines[2].NoNewline);
}
```

From [`tests/diff.test.osy`](tests/diff.test.osy).

```osy
using Osysharp.Diff;

[Test]
void a_binary_file_is_a_note_and_never_its_bytes() {
  var f = ParseDiff(MultiFilePatch())[2];
  Assert.True(f.Binary);
  Assert.Equal(DiffChange.Added, f.Change);
  Assert.Equal("assets/logo.png", DiffPathOfFile(f));
  Assert.Equal(0, f.Hunks.Count);
  Assert.Equal("Binary file: its contents are not shown.", f.Notes[0]);
  var notes = Kind(RowsOf(MultiFilePatch(), new List<string>(), 1000), DiffRowKind.Note);
  Assert.True(notes.Any(n => n.Text == "Binary file: its contents are not shown." && n.File == 2));
}
```

From [`tests/diff.test.osy`](tests/diff.test.osy).

## Learn more

- [The Osysharp.Diff reference](https://osysharp.com/reference/ui/diff-kit/)
- [Osysharp.Diff in the package catalogue](https://osysharp.com/packages/#osysharp-diff)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
