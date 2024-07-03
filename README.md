# Resumen del proyecto 

# Introducción al SQL

## Descripción del Proyecto

En este proyecto se debe resolver el misterio de un asesinato ocurrido el 15/01/2018, donde solamente tenemos una pista que ocurrio en SQL CIity.


## Propósito de este proyecto

A través de juego de busqueda del asesino, el objetivo de  este proyecto es poner en práctica los conocimientos de consultas SQL y con las pistas obtenidas dar con el culpable.
Para esto contamos con la base de datos de la policia para hacer las consultas.

## Pasos para dar con el Asesino.

*  Acceder a la Base de Datos

Se accedió a la base de datos de la policía de SQL City utilizando la plataforma SQL Murder Mystery. Donde ya contabamos con los datos de que el asesinato ocurrió el dia 15/01/2018 y que ocurrio en la ciudad de SQL City.

* Consultar en la base de datos que ya sabemos: 

SELECT *
FROM crime_scene_report
WHERE type='murder' AND date=20180115

Dando como resultado: 

index	date	type	description	city
0	20180115	murder	Life? Dont talk to me about life.	Albany
1	20180115	murder	Mama, I killed a man, put a gun against his head...	Reno
2	20180115	murder	Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".	SQL City

Donde nos interesa es lo que sucedió en SQL City.

* Con esto, pude serguir investigando sobre estos 2 testigos


SELECT id, name,address_street_name
FROM person
WHERE name LIKE 'Annabel%' AND address_street_name= 'Franklin Ave'

id	name	address_street_name
0	16371	Annabel Miller	Franklin Ave

En esta consulta obtuve el ID de annabel y asi poder conocer su testimonio.

SELECT transcript
FROM interview
WHERE person_id=16371

index	transcript
0	I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

Se puede conocer la fecha en la cual ella lo reconoció en el gym.

* Se consulta al segundo testigo , la cual indicaba que vivia en la ultima casa de Northwestern Dr. y Se ordeno por dirección descendente.

SELECT *
FROM person
WHERE address_street_name= 'Northwestern Dr'
ORDER BY address_number DESC

	id	   name 	       license_id	  address_number	      address_street_name	     ssn

14887	Morty Schapiro	   118009	       4919	                  Northwestern Dr	       111564949
17729	Lasonya Wildey	   439686	       3824	                  Northwestern Dr	       917817122
53890	Sophie Tiberio	   957671	       3755	                  Northwestern Dr	       442830147

* Con el ID de este testigo se puede consultar su testimonio 

SELECT transcript
FROM interview
WHERE person_id=14887

index	transcript
0	I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".


SELECT dri.id, dri.plate_number, per.name, per.address_street_name
FROM drivers_license dri
JOIN person per ON dri.id= per.license_id
WHERE dri.plate_number LIKE 'H42W%'

id	plate_number	name	address_street_name
0	183779	H42W0X	Maxine Whitely	Fisk Rd


SELECT *
FROM get_fit_now_check_in
WHERE membership_id LIKE '48Z%'AND check_in_date=20180109


membership_id	check_in_date	check_in_time	check_out_time
48Z7A	        20180109	            1600	         1730
48Z55	        20180109	            1530	         1700

Consultando la membresia que dijo el testigo con la fecha vista por la otra persona, me arroja 2 personas que son sospechosas.

* Para ir descartando se hizo otras consultas 

SELECT *
FROM get_fit_now_member get
JOIN get_fit_now_check_in  get_check ON get.id= get_check.membership_id
WHERE get_check.membership_id IN('48Z7A', '48Z55')

id	    person_id	name	membership_start_date	membership_status	membership_id	check_in_date	check_in_time	check_out_time
48Z7A	28819	  Joe Germuska	    20160305	   gold	                48Z7A	            20180109	   1600	          1730
48Z55	67318	  Jeremy Bowers	    20160101	   gold	                48Z55	            20180109	   1530	          1700


SELECT *
FROM facebook_event_checkin face
JOIN person per ON face.person_id= per.id
WHERE per.id IN ('28819','67318')

person_id	event_id	event_name	date	id	name	license_id	address_number	address_street_name	ssn
0	67318	4719	The Funky Grooves Tour	20180115	67318	Jeremy Bowers	423327	530	Washington Pl, Apt 3A	871539279
1	67318	1143	SQL Symphony Concert	20171206	67318	Jeremy Bowers	423327	530	Washington Pl, Apt 3A	871539279

En esta consulta me indica que solo Jeremy Bowers tiene evento en su face el dia 15.01.2018, el dia del asesinado. No hay nada de Joe Germuska.

check_suspect("Jeremy Bowers")

Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge,try querying the interview transcript of the murderer to find the real villain behind this crime.If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries.
Use this same `check_suspect` function with your new suspect to check your answer.
True


Para ir más alla de encontrar el asesino, quise buscar su testimonio en la base de datos y encontrar ¿Que lo llevó a realizar el crimen?


* Con el id, se ingresó  a la entrevista y se obtuvo lo siguiente:

SELECT transcript
FROM interview
WHERE person_id=67318

index	transcript
0	I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

Aqui informa que una mujer de mucho dinero fue quien lo contrato para realizar el asesinato  y dió detalles de esta persona.

SELECT *
FROM facebook_event_checkin
WHERE event_name= 'SQL Symphony Concert' And date BETWEEN 20171201 AND 20171231
ORDER BY person_id ASC

person_id	event_id	event_name	date
0	11173	1143	SQL Symphony Concert	20171223
1	19260	1143	SQL Symphony Concert	20171214
2	19292	1143	SQL Symphony Concert	20171213
3	24397	1143	SQL Symphony Concert	20171208
4	24556	1143	SQL Symphony Concert	20171207
5	24556	1143	SQL Symphony Concert	20171221
6	24556	1143	SQL Symphony Concert	20171224
7	28582	1143	SQL Symphony Concert	20171220
8	28582	1143	SQL Symphony Concert	20171215
9	43366	1143	SQL Symphony Concert	20171207
10	58898	1143	SQL Symphony Concert	20171220
11	62596	1143	SQL Symphony Concert	20171225
12	67318	1143	SQL Symphony Concert	20171206
13	69325	1143	SQL Symphony Concert	20171206
14	69699	1143	SQL Symphony Concert	20171214
15	79312	1143	SQL Symphony Concert	20171203
16	81526	1143	SQL Symphony Concert	20171202
17	92343	1143	SQL Symphony Concert	20171212
18	99716	1143	SQL Symphony Concert	20171206
19	99716	1143	SQL Symphony Concert	20171212
20	99716	1143	SQL Symphony Concert	20171229

Obteniendo a 2 personaas que fueron a ese evento 3 veces en esa fecha mencionada.

* se consulta en la base de dato

SELECT *
FROM person
WHERE id IN( '24556' , '99716')


id	name	license_id	address_number	address_street_name	ssn
0	24556	Bryan Pardo	101191	703	Machine Ln	816663882
1	99716	Miranda Priestly	202298	1883	Golden Ave	987756388

Aqui se descartó al hombre ya que el asesino habia dicho que fue una mujer.


SELECT id, hair_color,height, plate_number,car_make, car_model
FROM drivers_license
WHERE id= '202298'

id	hair_color	height	plate_number	car_make	car_model
0	202298	red	66	500123	Tesla	Model S

Aqui se puede observa que si es la persona que describe el asesino pero no hay más para consultar.

## Conclusiones

Tras analizar las evidencia de toda estas consulta, fue de gran satisfacción dar con el culpable, encontrar evidencias de lo ocurrido y poner en práctica los conocimientos SQL.




