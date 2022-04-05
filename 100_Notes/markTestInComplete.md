#php/laravel/test 

`$this->markInTestComplete()` を使うとテストをスキップできます。  
スキップしたい時もあるでしょうが、割れ窓になる可能性があります。  
もしどうしてもスキップしたい時は、スキップすることになった変更の内容のコミットへのリンクなどを@refとかで貼っておくとテストがメンテしやすくなります。  
メンテされないままで放置されるテストは書く意味がほとんどなく、無駄です。

---

- なるべくSkipはしない
- どうしてもSkipしたい場合は**理由を必ず書く**