# 層化否定

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/stratified-negation.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/stratified-negation.md)ください。

Flix は、ルール本体で否定を制限付きで使用できるようにする _層化否定(stratified negation)_ をサポートしています。例えば：

```flix
def main(): Unit \ IO =
    let movies = #{
        Movie("The Hateful Eight").
        Movie("Interstellar").
    };
    let actors = #{
        StarringIn("The Hateful Eight", "Samuel L. Jackson").
        StarringIn("The Hateful Eight", "Kurt Russel").
        StarringIn("The Hateful Eight", "Quentin Tarantino").
        StarringIn("Interstellar", "Matthew McConaughey").
        StarringIn("Interstellar", "Anne Hathaway").
    };
    let directors = #{
        DirectedBy("The Hateful Eight", "Quentin Tarantino").
        DirectedBy("Interstellar", "Christopher Nolan").
    };
    let rule = #{
        MovieWithoutDirector(title) :-
            Movie(title),
            DirectedBy(title, name),
            not StarringIn(title, name).
    };
    query movies, actors, directors, rule
        select title from MovieWithoutDirector(title) |> println
```

このプログラムは、映画・俳優・監督に関する情報を保持する 3 つのローカル変数を定義しています。ローカル変数 `rule` には、監督がその映画に出演していないすべての映画を捉えるルールが含まれています。このルールで否定が使われている点に注目してください。クエリは、文字列 `"Interstellar"` を含む配列を返します。これは、Christopher Nolan がその映画に出演していないためです。

> **注意:** Flix は、プログラムが層化されていること、すなわち、否定が使用されている箇所に再帰的な依存関係が存在しないことを強制します。もし存在する場合、Flix コンパイラはそのプログラムを拒否します。

<!--
# Stratified Negation

Flix supports _stratified negation_ which allow
restricted use of negation in rule bodies.
For example:

```flix
def main(): Unit \ IO =
    let movies = #{
        Movie("The Hateful Eight").
        Movie("Interstellar").
    };
    let actors = #{
        StarringIn("The Hateful Eight", "Samuel L. Jackson").
        StarringIn("The Hateful Eight", "Kurt Russel").
        StarringIn("The Hateful Eight", "Quentin Tarantino").
        StarringIn("Interstellar", "Matthew McConaughey").
        StarringIn("Interstellar", "Anne Hathaway").
    };
    let directors = #{
        DirectedBy("The Hateful Eight", "Quentin Tarantino").
        DirectedBy("Interstellar", "Christopher Nolan").
    };
    let rule = #{
        MovieWithoutDirector(title) :-
            Movie(title),
            DirectedBy(title, name),
            not StarringIn(title, name).
    };
    query movies, actors, directors, rule
        select title from MovieWithoutDirector(title) |> println
```

The program defines three local variables that
contain information about movies, actors, and
directors.
The local variable `rule` contains a rule that
captures all movies where the director does not star
in the movie.
Note the use negation in this rule.
The query returns an array with the string
`"Interstellar"` because Christopher Nolan did not
star in that movie.

> **Note:** Flix enforces that programs are stratified, i.e. a program must not
> have recursive dependencies on which there is use of negation. If there is,
> the Flix compiler rejects the program.
-->
