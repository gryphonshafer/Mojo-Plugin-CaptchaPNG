# NAME

Mojolicious::Plugin::CaptchaPNG - PNG captcha generation and validation Mojolicious plugin

# VERSION

version 1.06

[![test](https://github.com/gryphonshafer/Mojo-Plugin-CaptchaPNG/workflows/test/badge.svg)](https://github.com/gryphonshafer/Mojo-Plugin-CaptchaPNG/actions?query=workflow%3Atest)
[![codecov](https://codecov.io/gh/gryphonshafer/Mojo-Plugin-CaptchaPNG/graph/badge.svg)](https://codecov.io/gh/gryphonshafer/Mojo-Plugin-CaptchaPNG)

# SYNOPSIS

    # Simple Mojolicious
    $app->plugin( CaptchaPNG => { ttf => 'font.ttf' } );

    my $captcha_value = $app->get_captcha_value;
    my $success = $app->check_captcha_value($captcha_value);
    $app->clear_captcha_value;

    # Customized Mojolicious
    $app->plugin( CaptchaPNG => {
        routes      => $app->routes,
        method      => 'any',
        path        => '/captcha',
        key         => '_plugin_captchapng',
        width       => 230,
        height      => 50,
        ttf         => 'font.ttf',
        size        => 20,
        rotation    => 8,
        x           => 20,
        y_base      => 35,
        y_rotate    => 100,
        noise       => 1_250,
        background  => [ 255, 255, 255 ],
        text_color  => [ 'urand(128)', 'urand(128)', 'urand(128)' ],
        noise_color => [ 'urand(128) + 128', 'urand(128) + 128', 'urand(128) + 128' ],
        value       => sub {
            return int(
                Mojolicious::Plugin::CaptchaPNG::urand(
                    10_000_000 - 1_000_000
                )
            ) + 1_000_000;
        },
        display => sub {
            my ($display) = @_;
            $display =~ s/^(\d{2})(\d{3})/$1-$2-/;
            $display =~ s/(.)/ $1/g;
            return $display;
        },
    } );

    # Mojolicious::Lite
    plugin( CaptchaPNG => { ttf => 'font.ttf' } );

# DESCRIPTION

This module is a Mojolicious plugin for basic image captcha generation and
validation.

During registration (when `plugin` is called), the plugin will setup a route
(which defaults to `/captcha`) that will respond with a generated PNG captcha.
The image is generated using [GD::Image](https://metacpan.org/pod/GD%3A%3AImage). The plugin will also setup helper
methods to get, check, and clear the captcha value, which is stored in the
session.

# SETTINGS

Much of the plugin's settings are customizable, but only 1 is required.

## ttf

This is the only setting that's required. It is the path to a TTF font file that
will be used to generate the text in the captcha image.

## routes

This a [Mojolicious::Routes](https://metacpan.org/pod/Mojolicious%3A%3ARoutes) object. If not set, it defaults to:

    $app->routes

## method

The value to use when setting up the route method. If not set, it defaults to
`any`.

## path

The value to use when setting up the route path. If not set, it defaults to
`/captcha`.

## key

When a captcha image is generated, the value of the captcha text is stored in
the session under this key. If not set, it defaults to `_plugin_captchapng`.

## width, height

The width and height of the generated captcha image. If not set, these default
to `230` and `50` respectively.

## size

The font size of the text in the captcha image. If not set, it defaults to `20`.

## rotation

The amount of rotation to be made to the text in the captcha image. If not set,
it defaults to `8`.

## x

The `x` coordinate [GD::Image](https://metacpan.org/pod/GD%3A%3AImage) uses for the text. If not set, it defaults to
`20`.

## y\_base

The base value used for `y` in [GD::Image](https://metacpan.org/pod/GD%3A%3AImage) for the text. If not set, it
defaults to `35`.

## y\_rotate

The rotational value used for `y` in [GD::Image](https://metacpan.org/pod/GD%3A%3AImage) for the text. If not set, it
defaults to `100`.

## noise

The amount of noise to generate in the image. If not set, it defaults to
`1_250`.

## background

An array reference of 3 expressions, which will be used to set the color of the
background in the image. Values will be evaluated before used. If not set, it
defaults to:

    [ 255, 255, 255 ]

## text\_color

An array reference of 3 expressions, which will be used to set the color of the
text in the image. Values will be evaluated before used. If not set, it defaults
to:

    [ 'urand(128)', 'urand(128)', 'urand(128)' ]

Note that `urand` is provided by this library. See below.

## noise\_color

An array reference of 3 valexpressionses, which will be used to set the color of
the noise color in the image. Values will be evaluated before used. If not set,
it defaults to:

    [ 'urand(128) + 128', 'urand(128) + 128', 'urand(128) + 128' ]

Note that `urand` is provided by this library. See below.

## value

A subroutine reference that will be called to generate the value used for the
text of the captcha. If not set, it defaults to:

    sub {
        return int(
            Mojolicious::Plugin::CaptchaPNG::urand(
                10_000_000 - 1_000_000
            )
        ) + 1_000_000;
    }

## display

A subroutine reference that will be called and passed a value. The subroutine
is expected to alter the value for display purposes. For example, adding spaces
or dashes or other such things. If not set, it defaults to:

    sub {
        my ($display) = @_;
        $display =~ s/^(\d{2})(\d{3})/$1-$2-/;
        $display =~ s/(.)/ $1/g;
        return $display;
    }

# HELPER METHODS

## get\_captcha\_value

This method (expected to be used in a [Mojolicious](https://metacpan.org/pod/Mojolicious) controller) will return the
stored captcha value from the most recent image generation.

    my $captcha_value = $app->get_captcha_value;

## set\_captcha\_value

This method (likely never used, but if used would be expected to be used in a
[Mojolicious](https://metacpan.org/pod/Mojolicious) controller) will set a captcha value.

    $app->set_captcha_value(42);

## check\_captcha\_value

This method (expected to be used in a [Mojolicious](https://metacpan.org/pod/Mojolicious) controller) expects a
captcha value and will return true or false if it matches the stored captcha
value.

    my $success = $app->check_captcha_value($captcha_value);

On success, the captcha value is removed from the session.

## clear\_captcha\_value

Removes the captcha value from the session.

    $app->clear_captcha_value;

# OTHER METHOD

## urand

This method is a functional replacement of the core `rand` but using
[Crypt::URandom](https://metacpan.org/pod/Crypt%3A%3AURandom) for randomness.

# SEE ALSO

[Mojolicious](https://metacpan.org/pod/Mojolicious), [Mojolicious::Plugin](https://metacpan.org/pod/Mojolicious%3A%3APlugin).

You can also look for additional information at:

- [GitHub](https://github.com/gryphonshafer/Mojo-Plugin-CaptchaPNG)
- [MetaCPAN](https://metacpan.org/pod/Mojolicious::Plugin::CaptchaPNG)
- [GitHub Actions](https://github.com/gryphonshafer/Mojo-Plugin-CaptchaPNG/actions)
- [Codecov](https://codecov.io/gh/gryphonshafer/Mojo-Plugin-CaptchaPNG)
- [CPANTS](http://cpants.cpanauthors.org/dist/Mojo-Plugin-CaptchaPNG)
- [CPAN Testers](http://www.cpantesters.org/distro/M/Mojo-Plugin-CaptchaPNG.html)

# AUTHOR

Gryphon Shafer <gryphon@cpan.org>

# COPYRIGHT AND LICENSE

This software is Copyright (c) 2025-2050 by Gryphon Shafer.

This is free software, licensed under:

    The Artistic License 2.0 (GPL Compatible)
