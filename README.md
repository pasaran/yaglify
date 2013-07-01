# yaglify

После компиляции [yate](https://github.com/pasaran/yate)-шаблонов, получающийся javascript не до конца оптимален.
Например, там часто встречаются такие фрагменты:

    var r0 = '';

    r0 += closeAttrs(a0);
    r0 += '<div class="' + 'hello' + '">';
    r0 += '<span class="' + 'foo' + '">';
    r0 += nodeset2xml( selectNametest('hello', c0, []) );
    r0 += '</span>';
    r0 += '</div>';

Хочется написать некий постпроцессор, который сделает из этого кода такой:

    var r0 = '' + closeAttrs(a0) + '<div class="hello"><span class="foo">' + nodeset2xml( selectNametest('hello', c0, []) ) + '</span></div>';

К сожалению, существующие компрессоры javascript'а (например, `uglifyjs`) не справляются с этой задачей.
Например, `uglifyjs` не может оптимизировать `var s = x + 'foo' + 'bar';` до `var s = x + 'foobar';`.

При этом нет задачи написать полноценный компрессор, который заменит тот же `uglifyjs`.
Хочется особо сложные и часто встречающиеся случаи (именно в сгенеренном коде) поправить,
а остальное уже оставить для `uglifyjs`.

## Инсталляция

    npm install yaglify

## Использование

    yaglify foo.js

## Что умеет делать:

  * Заменяет:

        r += expr1;
        r += expr2;
        ...
        r += exprN;

    на:

        r = r + expr1 + expr2 + ... + exprN;

  * Заменяет:

        r = expr0;
        r += expr1;
        r += expr2;
        ...
        r += exprN;

    на:

        r = expr0 + expr1 + expr2 + ... + exprN;

  * Заменяет:

        var r = expr0;
        r += expr1;
        r += expr2;
        ...
        r += exprN;

    на:

        var r = expr0 + expr1 + expr2 + ... + exprN;

  * Заменяет:

        'foo' + 'bar'

    на

        'foobar'

  * Заменяет:

        ( x + ( y + ( z + 'foo' ) + 'bar'

    на

        x + ( y + ( z + 'foobar' )

  * И кое-какие другие оптимизации, связанные с ассоциативностью сложения.

