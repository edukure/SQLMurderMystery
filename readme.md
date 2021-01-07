# SQL Murder Mystery

A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.

```SQL
SELECT sql 
  FROM sqlite_master
  WHERE name = 'crime_scene_report'
```
`CREATE TABLE crime_scene_report ( date integer, type text, description text, city text )`

```SQL
SELECT date, description
    FROM crime_scene_report
    WHERE city = "SQL City" and type = "murder"
```
DATE: 20180115 -> date of murder

DESCRIPTION: Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

## Information about victims

### Info about person table:
```SQL
SELECT sql 
    FROM sqlite_master
    WHERE name = 'person'
```

`CREATE TABLE person ( id integer PRIMARY KEY, name text, license_id integer, address_number integer, address_street_name text, ssn integer, FOREIGN KEY (license_id) REFERENCES drivers_license(id) )`

### Info about 1st witness
```SQL
SELECT name, license_id, address_number, address_street_name, ssn 
    FROM person
    WHERE address_street_name = "Northwestern Dr"
    ORDER BY address_number DESC
    LIMIT 1
```
- **id:** 14887
- **name:** Morty Schapiro
- **license_id:** 118009	
- **address_number:** 4919
- **address_street_name:** Northwestern Dr
- **ssn:** 111564949

### Info about 2nd witness

```SQL
SELECT id, name, license_id, address_number, address_street_name, ssn 
    FROM person
    WHERE name LIKE "Annabel%" AND address_street_name = "Franklin Ave"
```
- **id:** 16371
- **name:** Annabel Miller
- **license_id:**	490173
- **address_number:**	103	
- **address_street_name:** Franklin Ave	
- **ssn:** 318771143

## Interviews

### Interview table
```SQL
SELECT sql 
    FROM sqlite_master
    WHERE name = 'interview'
```

`CREATE TABLE interview ( person_id integer, transcript text, FOREIGN KEY (person_id) REFERENCES person(id) )`

### Morty Shapiro
```SQL
SELECT person_id, transcript 
    FROM interview
    WHERE person_id = 14887
```

**transcript**: I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

### Annabel Miller
```SQL
SELECT person_id, transcript 
    FROM interview
    WHERE person_id = 16371
```

**transcript**: I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

## How to get to the murderer (1):
-  get who was at the Gym at January 9th with bag starting with 48Z from table get_fit_now_check_in
- find matching ids from get_fit_now_member table and create a names list
- look up drivers_license table for matching plate and create a list of license_ids
- look up person table for license_ids and match the responses to the names list

### Getting info from get_fir_now_check_in table
```SQL
SELECT sql 
    FROM sqlite_master
    WHERE name = 'get_fit_now_check_in'

```
`CREATE TABLE get_fit_now_check_in ( membership_id text, check_in_date integer, check_in_time integer, check_out_time integer, FOREIGN KEY (membership_id) REFERENCES get_fit_now_member(id) )`

### Getting membership_id from people that checked in at given date and that starts with 48Z
```SQL
SELECT membership_id 
    FROM get_fit_now_check_in
    WHERE check_in_date = 20180109 AND membership_id LIKE "48Z%"
```

`48Z7A 48Z55`

### Getting names and person_id from get_fit_now_member

```SQL
SELECT name, person_id 
    FROM get_fit_now_member
    WHERE id = "48Z55" OR id = "48Z7A"
```
**name:** Jeremy Bowers    **person_id:** 67318

**name:** Joe Germuska    **person_id:** 28819

### license_id from matching plates
```SQL
SELECT id
	FROM drivers_license
	WHERE plate_number LIKE "%H42W%"
```
`183779 423327 664760`


### finding names from license_id
```SQL
SELECT name
  FROM person
  WHERE license_id = 183779 OR license_id = 423327 OR license_id = 664760
```

`Tushar Chandra, Jeremy Bowers, Maxine Whitely`


## How to get to the murderer (2):
- join get_fit_now_check_in, get_fit_now_member, person and drivers_license tables
- look up clues (plate number, check in date and membership id) to find the culprit's name

```SQL
SELECT p.name, plate_number, fnm.id as fit_now_id, check_in.check_in_date
FROM person p
JOIN drivers_license d
  ON p.license_id = d.id 
JOIN get_fit_now_member fnm
  ON fnm.person_id = p.id
JOIN get_fit_now_check_in check_in
  ON check_in.membership_id = fnm.id
WHERE plate_number LIKE "%H42W%" AND fnm.id LIKE "48Z%" AND check_in.check_in_date = 20180109
```

> Jeremy Bowers is the killer!
