
django-docimport is a small framework for importing bulk data into your Django application from document formats.

It was written to address a specific problem that the author runs into frequently:

 * Data exists in a known document format (whether RSS, JSON, CSV, INI, an ad-hoc text format, etc)
 * You want to get that data into a structured relational form that you have defined in a Django model
 * The data documents are append-only; new data may be added, but once it's there, it doesn't go away
 * When new data is appended to a document, the data document should be reimported; newly added data 
   should be inserted, but data already in the relational database should not be duplicated there
 * Optionally, existing data may change between imports. You may want to update existing records with
   its new values if you can identify that it is still the same data referent.

Conceptually this is somewhat similar to running `svn up`: the Django database is your working copy,
and the data document represents the repository's current state.

What django-docimport doesn't do:

 * It does not know how to import data from any given format. You must provide the logic to parse
   a document and translate it into a relational model.

 * Likewise, it does not define any data models.

 * It does not update records automatically. You can build this on top of django-docimport however
   is best suited for your application -- manual, cron, commit hooks, etc.

 * Likewise, it does not know anything about the documents being imported, or where they come from.

Some of the specific applications that drove the author to write this:

 1. A client has a large set of records in an Excel spreadsheet that must be transferred to a Django
    project. The records are provided as a CSV export. The records have some minor errors in them,
    which are easy to spot after they are imported into the Django project. So records are imported
    into a development database and checked for errors; errors are corrected in the CSV file; the
    CSV file is then reimported into a clean database. When the project is done, the Django database
    is the canonical source for the data, and the CSV and Excel spreadsheets are discarded.

 2. The author and a friend send text messages to each other often. We want to analyze our texts in
    various ways -- frequency over time, occurance of specific words, etc. Thanks to the iPhone, we
    have a full archive of texts going back two years. The iPhone can export this archive to an XML
    format, which we can then parse and import into a Django project for analysis. Every so often we
    will re-export the archive and reimport it, to update our analysis with the latest texts.

 3. Similar to #2, but this time the data comes from a blog hosted on Blogger. Blogger allows you to
    export your entire blog archive in an XML-based "Blogger export file."

 4. flunc-webrunner executes flunc tests using variables from HTTP (POST /first/testsuite/?user=joe)
    and should serve HTML forms that expose those variables. Flunc itself doesn't maintain knowledge
    about what variables are available for a given test suite. That's actually a sort of relational
    metadata on the tests. So it's a natural fit for a django-vcexport app: flunc-webmanager exports
    static HTML web forms, that can be served directly, which represent its relational analysis about
    the tests and their variables.

    So again here's the same "import a set of canonical documents into Django for analysis" pattern.
    We want to write a little import script that figures out the variables used in a test, given that
    test file. (Individual .twill tests define variables. .tsuite suites can pass in their own variables,
    and suites can contain both suites and tests. But if we just figure out the direct variables used
    in each test or suite, the relations that we build up during import can construct the full variables
    available for a given suite.)

    With django-docimport, we could hook up a custom "flunc" importer to a checkout of ftests. 
