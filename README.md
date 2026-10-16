# humanize

Human-readable formatting for SSPUR: byte sizes in SI and IEC units, durations (compact, in words and approximate), relative times ("3 minutes ago"), ordinals, English plurals and singulars, lists ("a, b and c"), compact numbers ("1.2k"), thousands separators, numbers in words, Roman numerals, percentages, fractions, scientific notation and SI prefixes. Byte sizes, durations and compact numbers parse back too.

Everything is pure and English. The rounding rules are stated on each function (rounding can carry into the next unit: 999950 bytes is "1 MB", not "1000 kB"). For locale-specific number and date formats, use the builtin `Locale` (`locale("de-DE")` and friends).

## Example

```sspur
use humanize.{format_bytes, format_bytes_iec, parse_bytes, format_duration, format_duration_long, relative_time, parse_duration, ordinal, quantity, join_truncated, compact, number_words, to_roman, fraction}

fn main() -> Unit ! log
= do
  log("{format_bytes(1536000)} / {format_bytes_iec(1536000)} / {parse_bytes("1.5 GB")}")
  log(format_duration(millis(3723250)))
  log(format_duration_long(millis(3723000)))
  log(relative_time(mins(-3)))
  log("{parse_duration("1h30m").map(_.mins)} minutes")
  log("{ordinal(22)} {quantity(0, "file")}, {quantity(3, "person")}")
  log(join_truncated(["ana", "ben", "cai", "dee", "eve"], 2))
  log("{compact(1234)} {compact(3400000)} {compact(999950)}")
  log(number_words(1042))
  log("{to_roman(2026)} {fraction(2.75, 8)}")
```

```console
1.5 MB / 1.5 MiB / some(1500000000)
1h 2m 3s 250ms
1 hour, 2 minutes and 3 seconds
3 minutes ago
some(90) minutes
22nd no files, 3 people
ana, ben and 3 others
1.2k 3.4M 1M
one thousand forty-two
MMXXVI 2 3/4
```

## API

### Byte sizes

- `format_bytes(n)`: SI units (kB, MB, ... EB), one decimal, trailing zeros dropped: "1.5 kB".
- `format_bytes_iec(n)`: IEC units (KiB, MiB, ... EiB): "1.5 KiB".
- `format_bytes_with(n, binary, decimals)`: either, with up to `decimals` decimals.
- `parse_bytes(s)`: "1.5 GB", "10MiB", "512", "3 bytes" to bytes; K/KB are 1000, KiB 1024.

### Durations and relative times

- `format_duration(d)`, `format_ms(ms)`: "1d 2h 3m 4s", "1s 250ms", "0ms", "-1h 30m".
- `format_duration_long(d)`: "1 hour, 2 minutes and 3 seconds".
- `approximate_duration(d)`: Rails' distance_of_time_in_words: "less than a minute", "about 3 hours", "12 days", "over 2 years".
- `relative_time(d)`: for `d = t.since(now)`: "just now", "3 minutes ago", "in about 2 hours".
- `parse_duration(s)`: "1h30m", "90s", "1.5h", "2 days, 3 hours and 4 minutes", "-1h".

### Ordinals

- `ordinal(n)`: "1st", "2nd", "3rd", "11th", "21st", "111th", "-1st".
- `ordinal_suffix(n)`: just the suffix.

### Plurals

- `pluralize(word)`, `singularize(word)`: irregular nouns, uncountable nouns, the regular rules, and the word's case kept ("Box" to "Boxes").
- `plural_for(n, word)`: the word or its plural for a count.
- `count_noun(n, word)`: "1 file", "0 files", "1,234 files".
- `quantity(n, word)`: "no files", "1 file", "3 people".

### Lists

- `join_list(xs)`: "a, b and c". `join_list_oxford(xs)`: "a, b, and c". `join_or(xs)`: "a, b or c".
- `join_words(xs, conj, oxford)`: any conjunction.
- `join_truncated(xs, limit)`: "a, b and 3 others".

### Numbers

- `with_commas(n)`, `group_digits(n, sep)`, `with_commas_f64(x, decimals)`: "1,234,567", "1 234 567", "1,234.57".
- `compact(n)`, `compact_long(n)`, `compact_with(n, decimals, long)`: "1.2k", "3.4M", "1.2B", "1T"; "1.2 thousand". Rounded half away from zero.
- `parse_compact(s)`: "1.2k", "1.5 billion", "-1,500" back to an Int.

### Numbers in words

- `number_words(n)`: "one hundred twenty-three", "minus five"; the whole Int range.
- `ordinal_words(n)`: "first", "twenty-first", "one hundredth".
- `apnumber(n)`: "zero" to "nine", digits otherwise (AP style).

### Roman numerals

- `to_roman(n)`: 1 to 3999, "MCMXCIV".
- `from_roman(s)`: strict, only the standard form ("IIII" and "IC" are `none`).

### Percentages, fractions and magnitudes

- `percent(x, decimals)`, `percent_of(part, whole, decimals)`: "12.5%", "33.3%".
- `fraction(x, max_den)`: "1/2", "1 1/2", "-2 3/4", the closest fraction with a bounded denominator.
- `fraction_parts(x, max_den)`: the same as (numerator, denominator): (355, 113) for pi with 1000.
- `scientific(x, decimals)`: "1.23 x 10⁴".
- `metric(x, unit, decimals)`: SI prefixes from y to Y: "1.5 kV", "2 µF".

## Out of scope

- Languages other than English, and locale-specific separators beyond `group_digits`: the builtin `Locale` formats numbers (`format_int`, `format_f64`, `format_dec`) and dates in six locales.
- Calendar-aware phrases ("yesterday", "last Tuesday") and date formatting: see the `dates` package. This package describes lengths of time.
- A complete English morphology: `pluralize` and `singularize` cover the regular rules and a list of about a hundred irregular and uncountable nouns, matched as whole words (so "salesperson" becomes "salespersons"), like npm's `pluralize` but without user-added rules.
- British "and" in number words ("one hundred and five"), and words for numbers beyond Int (use `BigInt.str`).
- Durations in months or years ("2 months 3 days"), whose length depends on the calendar: see `dates.diff` and ISO 8601 durations there.

Compared with Python's `humanize` (naturalsize, naturaldelta, naturaltime, ordinal, intcomma, intword, apnumber, fractional, scientific, metric) and npm's `pretty-bytes`, `pretty-ms`, `pluralize` and `numeral`, the formatting functions are covered, plus parsers back for sizes, durations and compact numbers.

## Install

```
sspur add humanize
```

MIT licensed.
