# Home

For the source code visit the [GitHub Repository](https://github.com/materialsproject/MPCite).

For package instructions visit [PyPi](https://pypi.org/p/mp-cite).

# What is MPCite?

MPCite provides high-throughput, orchestrated functions for the Materials Project to interact programmatically with the DOE Office of Scientific and Technical Information (OSTI) via the E-Link API [[1]](https://github.com/doecode/elinkapi). It enables automated submission, validation, and management of metadata records and Digital Object Identifiers (DOIs) for materials data within the Materials Project ecosystem.

# What is E-Link?

Elink is 

# Implementation

MPCite's core functionalities are python functions which can be imported from the package. Altogether, they perform the afforementioned workflow.

## First, identify DOI entries to change...

```python
def find_out_of_date_doi_entries(
    rc_client: MongoClient,
    doi_client: MongoClient,
    robocrys_db: str,
    robocrys_collection: str,
    doi_db: str,
    doi_collection: str,
) -> list[OstiID]
```

## Next, update said DOI entries...

```python
def update_existing_osti_record(
    elinkapi: Elink, osti_id: OstiID, new_values: dict
) -> RecordResponse
```

```python
def update_state_of_osti_record(
    elinkapi: Elink, osti_id: OstiID, new_state="submit"
) -> RecordResponse
```

## If there are new records to be submitted...

```python
def submit_new_osti_record(
    elinkapi: Elink,
    new_record: Record,
    state="submit",
) -> RecordResponse
```

## And miscellaneous functions...

```python
def make_minimum_record_to_fully_release(
    title,  # required to make record
    product_type="DA",  # required to make record
    organizations=[
        Organization(type="RESEARCHING", name="LBNL Materials Project (LBNL-MP)"),
        Organization(
            type="SPONSOR",
            name="TEST SPONSOR ORG",
            identifiers=[{"type": "CN_DOE", "value": "AC02-05CH11231"}],
        ),
    ],  # sponsor org is necessary for submission
    persons=[Person(type="AUTHOR", last_name="Perrson")],
    site_ownership_code="LBNL-MP",
    access_limitations=["UNL"],
    publication_date=datetime.now().replace(
        hour=0, minute=0, second=0, microsecond=0
    ),  # what should this be?
    site_url="https://next-gen.materialsproject.org/materials",
) -> Record
```

One can delete OSIT records with a call to

```python
def delete_osti_record(
    elinkapi_token: str, osti_id: OstiID, reason: str
) -> RecordResponse
```

And additionally, there is a function to empty the review environment, which is handy for development and debugging.

```python
def emptyReviewAPI(reason, review_api)
```