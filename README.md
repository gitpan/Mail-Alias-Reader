# NAME

Mail::Alias::Reader - Read aliases(5) and ~/.forward declarations

# SYNOPSIS

    my $reader = Mail::Alias::Reader->open(
        'handle' => \*STDIN,
        'mode'   => 'aliases'
    );

    while (my ($name, $destinations) = $reader->read) {
        my @addresses = grep { $_->is_address } @{$destinations};

        print "$name: " . join(', ', map { $_->to_string } @addresses) . "\n";
    }

    $reader->close;

# DESCRIPTION

Mail::Alias::Reader is a small but oddly flexible module that facilitates the
reading of aliases(5)-style mail alias and ~/.forward declarations.  It does
not directly provide support for the writing and in-memory manipulation of
these files as a whole, however its limited feature set may be considered to
be a virtue.

The module can read mail aliases or ~/.forward declarations from a file, or
from an arbitrary file handle, as specified at the time of file reader
instantiation.  The destination objects returned are the same parser tokens
used internally, keeping code footprint low without being too much of a hassle.

# STARTING UP THE MODULE

- Mail::Alias::Reader->open(%opts)

    Open a file (`file`) or stream (`handle`) based on the values provided in
    `%opts`, returning a mail alias reader as a result.  A parsing `mode` can be
    supplied; by default, _aliases_ are expected, whereas a `mode` of _forward_
    can be specified as well.

# READING DECLARATIONS

- $reader->read()

    Seeks the current file stream for the next available declaration, and passes it
    through the parser, returning the data given by the parser, without any further
    manipulation.

    Depending on the parsing mode, the nature of the returned data will differ.
    Each of the following modes will cause `$reader->read()` to operate in the
    following manners:

    - `aliases`

        When Mail::Alias::Reader is set to read in `aliases` mode, a plain scalar
        value reflecting the name of the alias, followed by an `ARRAY` reference
        containing mail destinations, is returned, in list context.

            my ($name, $destinations) = $reader->read;

    - `forward`

            my $destinations = $reader->read;

        When Mail::Alias::Reader is set to read in `forward` mode, an `ARRAY`
        reference containing mail destinations is returned in a single scalar.

# THE MAIL DESTINATION TOKEN

Mail destination objects returned by `$reader->read()` are `HASH` objects
bless()ed into the Mail::Alias::Reader::Token package, and contain a small
handful of data attributes, but can be inspected with a variety of helper
functions in the form of instance methods.  Please see the
[Mail::Alias::Reader::Token](https://metacpan.org/pod/Mail::Alias::Reader::Token) documentation for a listing of these helper
functions.

The mail destination attributes include:

- `type`

    The type of token dealt with.  This can be one of _T\_ADDRESS_, _T\_COMMAND_,
    _T\_FILE_, or _T\_DIRECTIVE_.

    - _T\_ADDRESS_

        A mail destination token of type _T\_ADDRESS_ may indicate either a full mail
        address, or a local part.

    - _T\_COMMAND_

        A destination token of type _T\_COMMAND_ indicates that mail destined for the
        current alias is to be pipe()d to the specified command.

    - _T\_FILE_

        Any mail destined for the current alias will be appended to the file indicated
        by this destination token.

    - _T\_DIRECTIVE_

        Indicates any special destination in the format of `:directive:_argument_`.
        These are of course specific to the system's configured mail transfer agent.
        In this case, the name of the directive is captured in the token object's
        `name` attribute.

- `value`

    The textual value of the mail destination, parsed, cleansed of escape sequences
    that may have been present in the source file, containing only the data that
    is uniquely specified by the type of mail destination token given.  As an
    example, _T\_COMMAND_ destinations do not include the pipe ('|') symbol as a
    prefix; this is implied in the destination token type, rather.

- `name`

    Only appears in the prsence of a token typed _T\_DIRECTIVE_  When a mail alias
    destination in the form of `:directive:_argument_` is parsed, this contains
    the name of the '`directive`' portion.  Of course, the value in the
    '_argument_' portion is contained in the token's `value` field, but is
    considered optional, especially in the presence of a directive such as `:fail`.

# CLOSING THE STREAM

- $reader->close()

    Close the current file stream.  Any subsequent `$reader->read()` calls will
    return nothing.

# ERROR HANDLING

Carp::confess() is used internally for passing any error conditions detected
during the runtime of this module.

# AUTHOR

Written and maintained by Xan Tronix <xan@cpan.org>.

# COPYRIGHT

Copyright (c) 2014, cPanel, Inc.
All rights reserved.
http://cpanel.net/

This is free software; you can redistribute it and/or modify it under the same
terms as Perl itself.  See the LICENSE file for further details.
