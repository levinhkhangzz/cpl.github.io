---
title: "SQL Murder Mystery"
description: "Have fun investigating a murder in SQL City. you are a detective using SQL queries"
keywords: [technical, walktrough, sql, fun, game]
date: 2019-12-18
---

This post will not be much of a read, just my "logbook" from solving the SQL Murder Mystery today.

> Edit: Had to remove SQL results tables due to poor formatting of the theme. To see original "log" check out the [gist](https://gist.github.com/cpl/17943377f49d266ff4c8649fc5752adc).

**SPOILER** WARNING. This game was fun! I recommend it to any SQL amateur or pro. Give it a go at [https://mystery.knightlab.com/](https://mystery.knightlab.com/).

> There's been a Murder in SQL City! The SQL Murder Mystery is designed to be both a self-directed lesson to learn SQL concepts and commands and a fun game for experienced SQL users to solve an intriguing crime. 

> A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database. 

## Crime report

```sql
SELECT * FROM crime_scene_report
	WHERE type='murder' AND city='SQL City' AND date=20180115;
```

## Suspects

```sql
SELECT * FROM person
	WHERE name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave';
```

```sql
SELECT * FROM person
	WHERE address_street_name = 'Northwestern Dr'
	ORDER BY address_number DESC
  LIMIT 1;
```

## Suspects licenses

```sql
SELECT * FROM drivers_license WHERE id=490173 OR id=118009;
```

## Suspects GetFitNow Membership

```sql
SELECT get_fit_now_member.id, person_id, get_fit_now_member.name, membership_start_date, membership_status, person.name
	FROM get_fit_now_member
	JOIN person ON person.id = person_id
	WHERE person.id=16371 OR person.id=14887;
```

## Suspects GetFitNow CheckIn Times

```sql
SELECT * FROM get_fit_now_check_in WHERE membership_id = '90081';
```

## Suspects Interviews

```sql
SELECT * FROM interview WHERE person_id=16371 OR person_id=14887;
```

## Investigating transcripts

### Morty Schapiro's transcript

```sql
SELECT * FROM get_fit_now_member
	JOIN person ON person.id = person_id
	JOIN drivers_license ON drivers_license.id = person.license_id
	WHERE
		get_fit_now_member.id LIKE '48Z%' AND membership_status='gold' AND
		drivers_license.gender = 'male' AND drivers_license.plate_number LIKE '%H42W%';
```

```sql
SELECT * FROM get_fit_now_check_in
	WHERE membership_id = '48Z55';
```

```sql
SELECT * FROM interview
	WHERE person_id = 67318;
```

* The plot thickens
* We'll track this mysterious woman later on

```sql
SELECT * FROM facebook_event_checkin
	WHERE person_id = 67318;
```

### Annabel Miller's transcript

In her transcript she mentiones seeing the killer at the Gym. From her entry we know she was indeed there on 9th Jan, between `16:00` and `17:00`. This means the killer must have checked in before she left and the killer must have checked out after she left.

```sql
SELECT * FROM get_fit_now_check_in
	WHERE check_in_date=20180109
		AND check_in_time <  1700
		AND check_out_time > 1600;
```

* We have to remember that `90081` is *Annabel*
* Notice `48Z55` is *Jeremy Bowers*, the guy from Morty's transcript
* For `48Z7A` there is just the one check-in

```sql
SELECT * FROM get_fit_now_member
	JOIN person ON person_id = person.id
	WHERE get_fit_now_member.id = '48Z7A';
```

```sql
SELECT * FROM facebook_event_checkin
	WHERE person_id = 28819;
```

* This person attended no events

### Jeremy Bowers's 

> I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

* From this statement we can extract a few things
  * It's a `female`
  * `5'5" (65")` or `5'7" (67")`
  * Hair color is `red`
  * Car is `Tesla` `Model S`
  * She attended `SQL Symphony Concert 3`, 3 times in `December 2017`

```sql
SELECT * FROM person
	JOIN  drivers_license ON person.license_id = drivers_license.id
	WHERE
		drivers_license.gender = 'female'  AND
		drivers_license.hair_color = 'red' AND
		(height >= 65 AND height <= 67)    AND
		car_make = 'Tesla' AND car_model='Model S';
```

```sql
SELECT * FROM facebook_event_checkin
	WHERE person_id IN (78881, 90700, 99716);
```

## Investigating Miranda Priestly

* `99716`, *Miranda Priestly* is the only person matching the criteria
* Will investigate the other points just to get an idea of this person

```sql
SELECT * FROM income WHERE ssn=987756388;
```

```sql
SELECT * FROM get_fit_now_member WHERE person_id = 99716;
```

* Not a GetFitNow member

```sql
SELECT * FROM interview WHERE person_id = 99716;
```

* Did not give an interview

## Ending

Honestly not the ending I expected. I really hoped for this to go on for longer.
I've put in *Miranda Priestly* and **won**.

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
SELECT value FROM solution;
```
