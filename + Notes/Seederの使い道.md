### Seeder

- Seederに依存したテストケースはアンチパターン
- マスタデータのSeederは許可（書き換わることがない）
- Seedに依存していると無意識に順番依存性がある可能性がある
- 原則 Factoryにしたい

---

https://gist.github.com/bz0/ab5d8c3957c10e886bd9b8c2e2b02536

---