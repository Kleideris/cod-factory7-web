```sql
-- AIRPORT_DB ΑΣΚΉΣΕΙΣ:


-- A. Να γράψετε κατάλληλες εντολές SQL ώστε να απαντηθούν τα παρακάτω ερωτήματα:

-- 1. Στοιχεία των πτήσεων με ημερομηνία αναχώρησης την 01/05/2018 και προορισμό το Τορόντο.
SELECT *
FROM flights
WHERE depDate = '2018-05-01' AND toCity = 'Τορόντο';


-- 2. Στοιχεία των πτήσεων των οποίων η απόσταση κυμαίνεται μεταξύ 900 και 1500 μιλίων, ταξινομημένα με βάση την απόσταση σε αύξουσα διάταξη.
SELECT *
FROM flights
WHERE distance BETWEEN 900 AND 1500 
ORDER BY distance ASC;


-- 3. Συνολικός αριθμός των πτήσεων με ημερομηνία αναχώρησης μεταξύ 1/5/2018 μέχρι και 30/5/2018, ανά προορισμό.
SELECT COUNT(fno) AS Total_Flights, toCity
FROM flights
WHERE depDate BETWEEN '2018-05-01' AND '2018-05-30'
GROUP BY toCity;


-- 4. Προορισμοί και συνολικός αριθμό των πτήσεων ανά προορισμό, στους οποίους υπάρχουν τουλάχιστον τρείς πτήσεις.
SELECT toCity, COUNT(fno) AS TotalFlights
FROM flights
GROUP BY toCity
HAVING COUNT(fno) >= 3;


-- 5. Ονοματεπώνυμα των πιλότων που είναι πιστοποιημένοι στην λειτουργία τουλάχιστον τριών αεροσκαφών.
SELECT lastname, firstname, COUNT(E.empid) AS Number_Of_Certifications
FROM employees E 
JOIN certified C ON E.empid = C.empid
GROUP BY lastname, firstname
HAVING COUNT(E.empid) >= 3;


-- 6. Συνολικό κόστος των μηνιαίων μισθών όλων των υπαλλήλων της εταιρείας.
SELECT SUM(salary) AS Total_Cost_of_All_Employees
FROM employees;


-- 7. Συνολικό κόστος των μηνιαίων μισθών όλων των πιλότων της εταιρείας.
SELECT SUM(salary) AS Total_Cost_Of_Pilots
FROM (
    SELECT DISTINCT E.empid, E.salary
    FROM employees E
	JOIN certified C ON E.empid = C.empid
) AS Unique_Pilot_Salaries;


-- 8. Εμφανίστε το συνολικό κόστος των μηνιαίων μισθών των υπαλλήλων της εταιρείας που δεν είναι πιλότοι.
SELECT SUM(salary) AS Total_Cost_Of_Non_Pilot_Employee
FROM (
    SELECT E.empid, E.salary
    FROM employees E 
    LEFT JOIN certified C ON E.empid = C.empid
    WHERE C.empid IS NULL
) AS Unique_NonPilot_Salaries;


-- 9. Κατάλογος με τα ονόματα των αεροσκαφών που μπορούν να καλύψουν την πτήση από Αθήνα προς Μελβούρνη δίχως στάση για ανεφοδιασμό.
SELECT aname, crange
FROM aircrafts
WHERE crange >= ANY (
	SELECT distance
    FROM flights
    WHERE fromCity = 'Αθήνα' AND toCity = 'Μελβούρνη'
);


-- 10. Ονοματεπώνυμα των πιλότων που είναι πιστοποιημένοι στην λειτουργία αεροσκάφους τύπου Boeing (το όνομα του αεροσκάφους ξεκινάει με Boeing).
SELECT DISTINCT lastname, firstname
FROM employees E
JOIN certified C ON E.empid = C.empid
JOIN aircrafts A ON C.aid = A.aid
WHERE aname LIKE 'Boeing%'
GROUP BY lastname, firstname;


-- 11. Ονοματεπώνυμα των πιλότων που είναι πιστοποιημένοι σε αεροσκάφη με δυνατότητα πτήσης μεγαλύτερης των 3000 μιλίων, αλλά δεν είναι πιστοποιημένοι σε κανένα αεροσκάφος τύπου Boeing.
SELECT DISTINCT lastname, firstname
FROM employees E
JOIN certified C ON E.empid = C.empid
JOIN aircrafts A ON C.aid = A.aid
WHERE crange > 3000

EXCEPT

SELECT lastname, firstname
FROM employees E
JOIN certified C ON E.empid = C.empid
JOIN aircrafts A ON C.aid = A.aid
WHERE A.aname LIKE 'Boeing%';


-- 12. Ονοματεπώνυμα των υπαλλήλων με τον υψηλότερο μισθό.
SELECT E1.lastname, E1.firstname
FROM employees E1
LEFT JOIN employees E2 ON E1.salary < E2.salary
WHERE E2.salary IS NULL;


-- 13. Ονοματεπώνυμα των υπαλλήλων που έχουν τον δεύτερο υψηλότερο μισθό.
SELECT E1.lastname, E1.firstname
FROM employees E1
LEFT JOIN employees E2 ON E1.salary < E2.salary
GROUP BY E1.lastname, E1.firstname, E1.salary
HAVING COUNT(DISTINCT E2.salary) = 1;


-- 14. Ονόματα των αεροσκαφών για τα οποία όλοι οι πιστοποιημένοι στην λειτουργία τους πιλότοι έχουν μισθό τουλάχιστον 6000 ευρώ.
SELECT aname
FROM aircrafts A

EXCEPT

SELECT aname
FROM aircrafts A
JOIN certified C ON C.aid = A.aid
JOIN employees E ON C.empid = E.empid
GROUP BY aname, salary
HAVING salary < 6000;


-- 15. Κωδικoί πιλότων που είναι πιστοποιημένοι στην λειτουργία τουλάχιστον τριών αεροσκαφών, και το μεγαλύτερο εύρος πτήσης (crange) των αεροσκαφών στα οποία είναι πιστοποιημένοι.
SELECT E.empid, MAX(A.crange) AS Max_Cruise_Range
FROM employees E
JOIN certified C ON C.empid = E.empid
JOIN aircrafts A ON C.aid = A.aid
WHERE E.empid IN (
    SELECT empid
    FROM certified C
    GROUP BY C.empid
    HAVING COUNT(C.aid) >= 3
)
GROUP BY E.empid;


-- 16. Ονοματεπώνυμα των υπαλλήλων με μισθό μικρότερο από το κόστος της φθηνότερη πτήσης με προορισμό την Μελβούρνη.
SELECT E.firstname, E.lastname
FROM employees E
WHERE salary < (
	SELECT MIN(F.price)
	FROM flights F
	WHERE toCity = 'Μελβούρνη'
);


-- 17. Ονοματεπώνυμα και μισθός των υπαλλήλων που δεν είναι πιλότοι και κερδίζουν πάνω από τον μέσο όρο του μισθού των πιλότων.
SELECT E.firstname, E.lastname, salary
FROM employees E
LEFT JOIN certified C ON E.empid = C.empid
WHERE C.aid IS NULL
  AND salary > (
	SELECT AVG(E.salary)
	FROM employees E
	JOIN certified C ON E.empid = C.empid
);



-- B. ΟΨΕΙΣ

-- 18. Δημιουργεία όψη (pilots) που περιέχει όλα τα στοιχεία των πιλότων και όψη (others) με όλα τα στοιχεία των υπαλλήλων που δεν είναι πιλότοι.
CREATE VIEW  pilots AS
SELECT DISTINCT
	E.empid,
	E.lastname,
	E.firstname,
	E.salary
FROM employees E
JOIN certified C ON E.empid = C.empid;

CREATE VIEW  others AS
SELECT DISTINCT
	E.empid,
	E.lastname,
	E.firstname,
	E.salary
FROM employees E
LEFT JOIN certified C ON E.empid = C.empid
WHERE C.empid IS NULL;

-- 18.a Χρησιμοποιώντας τις όψεις που δημιουργήσατε, ξαναγράψτε τα ερωτήματα 7, 8 και 17.

-- 7. (Με χρήση Όψης) Συνολικό κόστος των μηνιαίων μισθών όλων των πιλότων της εταιρείας.
SELECT SUM(salary) AS Total_Cost_Of_Pilots
FROM pilots;

-- 8. (Με χρήση Όψης) Εμφανίστε το συνολικό κόστος των μηνιαίων μισθών των υπαλλήλων της εταιρείας που δεν είναι πιλότοι.
SELECT SUM(SALARY) AS Total_Cost_Of_NonPilots
FROM others;

-- 17. (Με χρήση Όψης) Ονοματεπώνυμα και μισθός των υπαλλήλων που δεν είναι πιλότοι και κερδίζουν πάνω από τον μέσο όρο του μισθού των πιλότων.
SELECT lastname, firstname, salary
FROM others
WHERE salary > (
	SELECT AVG(salary)
	FROM pilots
);


-- 19. Δημιουργεία όψης η οποία περιέχει τo όνομα κάθε αεροσκάφους και τα στοιχεία των πτήσεων (fno, fromCity, toCity) που το κάθε αεροσκάφος μπορεί να καλύψει δίχως ανεφοδιασμό.
CREATE VIEW aircraft_nonstop_flight_info AS
SELECT DISTINCT
	A.aname,
	F.fno,
	F.fromCity,
	F.toCity
FROM aircrafts A
JOIN flights F ON A.crange >= F.distance;

-- 19.a Χρησιμοποιώντας την όψη που δημιουργήσατε εμφανίστε έναν κατάλογο με τα ονόματα των αεροσκαφών και τον αριθμό των πτήσεων που κάθε αεροσκάφος μπορεί να εξυπηρετήσει.
SELECT aname, COUNT(*)
FROM aircraft_nonstop_flight_info
GROUP BY aname;



-- Γ. ΔΙΑΔΙΚΑΣΙΕΣ

-- 20. Δημιουργεία διαδικασίας η οποία θα εμφανίζει τον κωδικό κάθε πτήσης και δίπλα τον χαρακτηρισμό: "Φθηνή", "Κανονική" ή "Ακριβή".
-- Μία πτήση θεωρείται "φθηνή" αν το κόστος του εισιτηρίου είναι μέχρι και 500 ευρώ, "κανονική" αν το κόστος κυμαίνεται μεταξύ 501 και 1500 ευρώ και "ακριβή" αν το κόστος του εισιτηρίου ξεπερνάει τα 1500 ευρώ.
CREATE PROCEDURE Flight_Cost_Value
AS
BEGIN
	DECLARE @fno VARCHAR(50);
	DECLARE @price INT;
	DECLARE @costValue VARCHAR(50);

	DECLARE @results TABLE (
	fno VARCHAR(50),
	costValue VARCHAR(50)
	);

	DECLARE flights_cursor CURSOR FOR
	SELECT fno, price FROM flights;

	OPEN flights_cursor;
	FETCH NEXT FROM	flights_cursor INTO @fno, @price;

	WHILE @@FETCH_STATUS = 0
	BEGIN
		IF @price <= 500
		BEGIN
			SET @costValue = 'Φθηνή';
		END;

		ELSE IF @price BETWEEN 501 AND 1500
		BEGIN
			SET @costValue = 'Κανονική';
		END;

		ELSE
		BEGIN
			SET @costValue = 'Ακριβή';
		END;

		INSERT INTO @results (fno, costValue)
		VALUES (@fno, @costValue);

		FETCH NEXT FROM	flights_cursor INTO @fno, @price;
	END;

	CLOSE flights_cursor;
	DEALLOCATE flights_cursor;

	SELECT * FROM @results;
END;


-- 21. Δημιουργείστε μια διαδικασία η οποία θα δέχεται ως παραμέτρους το όνομα και τον κωδικό ενός πιλότου καθώς επίσης και το όνομα και τον κωδικό ενός αεροσκάφους. Η διαδικασία θα πιστοποιεί τον πιλότο στο συγκεκριμένο αεροσκάφος.
-- Αν ο πιλότος ή το αεροσκάφος δεν υπάρχουν στην βάση δεδομένων η διαδικασία θα πρέπει να τα εισαγάγει. Σε περίπτωση που ο πιλότος είναι ήδη πιστοποιημένος στην λειτουργία του συγκεκριμένου αεροσκάφους η διαδικασία θα εμφανίζει κατάλληλο μήνυμα.
CREATE PROCEDURE Certification_Procedure
	@lastname VARCHAR(50),
	@firstname VARCHAR(50),
	@empid INT,
	@aname VARCHAR(50),
	@aid INT
AS
BEGIN
	IF EXISTS (
		SELECT * FROM certified C
		JOIN employees E ON C.empid = E.empid
		WHERE C.empid = @empid
		AND C.aid = @aid
		AND E.lastname = @lastname
		AND E.firstname = @firstname
	)
	BEGIN
		PRINT 'Ο πιλότος είναι ήδη πιστοποιημένος γι αυτό το αεροσκάφος';
		RETURN;
	END;

	IF NOT EXISTS (
		SELECT * FROM employees
		WHERE lastname = @lastname
		AND firstname = @firstname
	)
	BEGIN
		PRINT 'Τα στοιχεία του πιλότου είναι λανθασμένα';
		RETURN;
	END;

	IF NOT EXISTS (SELECT * FROM employees WHERE empid = @empid)
	BEGIN
		INSERT INTO employees (empid, lastname, firstname)
		VALUES (@empid, @lastname, @firstname);
		PRINT 'Ο καινούριος πιλότος καταχωρήθηκε στη βάση';
	END

	IF NOT EXISTS (SELECT * FROM aircrafts WHERE aid = @aid)
	BEGIN
		INSERT INTO aircrafts (aid, aname) VALUES (@aid, @aname)
		PRINT 'Το καινούριο αεροσκάφος καταχωρήθηκε στη βάση';
	END;

	INSERT INTO certified (empid, aid) VALUES (@empid, @aid);
	PRINT 'Εγινε επιτυχής πιστοποίηση του πιλότου';
END;



-- Δ. ΠΥΡΟΔΟΤΕΣ

-- 22. Δημιουργείστε έναν πυροδότη ο οποίος θα ενεργοποιείται κάθε φορά που ένας πιλότος πιστοποιείται στην λειτουργία ενός αεροσκάφους. Αν με τη νέα πιστοποίηση ο πιλότος φθάνει τις τρείς, ο πυροδότης θα αυξάνει τον μισθό του κατά 10%.

CREATE TRIGGER Certification
ON certified
AFTER INSERT
AS
BEGIN
	DECLARE @empid INT;
	DECLARE @cert_counter INT;

	DECLARE cur CURSOR FOR
	SELECT empid from inserted;

	OPEN cur;
	FETCH NEXT FROM cur INTO @empid;
	
	WHILE @@FETCH_STATUS = 0
	BEGIN
		SET @cert_counter = (SELECT COUNT(DISTINCT aid) FROM certified WHERE empid = @empid);

		IF @cert_counter = 3
		BEGIN
			UPDATE employees
			SET salary = salary * 1.10
			WHERE empid = @empid;

		PRINT 'Increased Salary of pilot wth empid=' + CAST(@empid AS VARCHAR(10)) + ' by 10%';
		END;

		FETCH NEXT FROM cur INTO @empid;
	END;

	CLOSE cur;
	DEALLOCATE cur;
END;


-- 23. Δημιουργείστε ένα πυροδότη ο οποίος θα ενεργοποιείται κάθε φορά που ενημερώνεται η τιμή του εισιτηρίου μιας πτήσης. Ο πυροδότης θα καταγράφει στον πίνακα flight_history τις παρακάτω πληροφορίες:
-- * Κωδικό πτήσης (fno).
-- * Όνομα χρήστη που έκανε την ενημέρωση.
-- * Ημερομηνία και ώρα ενημέρωσης.
-- * Τιμή εισιτηρίου πριν την ενημέρωση.
-- * Τιμή εισιτηρίου μετά την ενημέρωση.

--Δημιουργεία πινακα flight_history.
CREATE TABLE flight_history (
	fno VARCHAR(4),
	usr_name VARCHAR(30),
	utime DATETIME NULL,
	price_old INT,
	price_new INT,
);

CREATE TRIGGER Ticket_Price_Update
ON flights
AFTER UPDATE
AS
BEGIN
	DECLARE
		@fno VARCHAR(4),
		@usr_name VARCHAR(30),
		@utime DATETIME,
		@price_old INT,
		@price_new INT;

	DECLARE cur CURSOR FOR
	SELECT i.fno, i.price AS new_price, d.price AS old_price
	FROM inserted i JOIN deleted d ON	i.fno = d.fno;

	OPEN cur;
	FETCH NEXT FROM cur INTO @fno, @price_new, @price_old;

	WHILE @@FETCH_STATUS = 0
	BEGIN
		IF @price_new <> @price_old
		BEGIN
			INSERT INTO flight_history (fno, usr_name, utime, price_old, price_new)
			VALUES (@fno, USER_NAME(), GETDATE(), @price_old, @price_new)

			PRINT 'Updated prices logged in table flight_history';
		END;

		FETCH NEXT FROM cur INTO @fno, @price_new, @price_old;
	END;

	CLOSE cur;
	DEALLOCATE cur;
END;