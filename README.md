# BooksOnline-CSAC45
Sharing the code with client 
DECLARE
record_loans gl_loans%ROWTYPE;
v_full_name varchar2(30);
v_interest_discount number(3,2) := 0.00;
v_new_interest_rate number(3,2) := 0.00;
v_balance gl_loans.loan_amount%TYPE;
v_month_counter number(4) := 0;
v_interest number(7,2) := 0.00;
v_monthly_payment gl_loans.monthly_payment%TYPE;
v_years number(3,1) := 0;
v_months number(3) := 0;
H1 varchar2(50) := 'Payment Schedule';
H1UL varchar2(50) := LPAD('-',LENGTH(H1),'-');

BEGIN
SELECT  * 
INTO record_loans
FROM gl_loans
WHERE loan_id = :ENTER_LOAN_ID;

-- Calculating Interest Rate Discount
CASE 
    WHEN record_loans.credit_score < 670 AND record_loans.credit_score >= 579 THEN v_interest_discount :=  0.25;
    WHEN record_loans.credit_score < 740 THEN v_interest_discount := 0.50;
    WHEN record_loans.credit_score < 800 THEN v_interest_discount := 0.75;
    WHEN record_loans.credit_score < 851 THEN v_interest_discount := 1.00;
END CASE;

-- Calculating New Annual Interest Rate
v_new_interest_rate := record_loans.annual_interest_rate - v_interest_discount;

v_balance := record_loans.loan_amount;

v_monthly_payment := record_loans.monthly_payment;

DBMS_OUTPUT.PUT_LINE(H1);
DBMS_OUTPUT.PUT_LINE(H1UL);
DBMS_OUTPUT.PUT_LINE('');

v_full_name := record_loans.FIRST_NAME ||' '|| record_loans.LAST_NAME;
DBMS_OUTPUT.PUT_LINE('Name: ' || v_full_name);
DBMS_OUTPUT.PUT_LINE('Loan Amount: ' || TO_CHAR(record_loans.loan_amount,'FM$999999.00'));
DBMS_OUTPUT.PUT_LINE('Annual Interest Rate: ' || record_loans.annual_interest_rate || '%');
DBMS_OUTPUT.PUT_LINE('Credit Score: ' || record_loans.credit_score || '  Interest Discount: '|| TO_CHAR(v_interest_discount,'FM$0.99') || '%');
DBMS_OUTPUT.PUT_LINE('New Annual Interest Rate: ' || v_new_interest_rate || '%');
DBMS_OUTPUT.PUT_LINE('Monthly Payment: ' || TO_CHAR(v_monthly_payment,'FM$9999.99'));
DBMS_OUTPUT.PUT_LINE('');

DBMS_OUTPUT.PUT_LINE('Month    Interest    Payment    Balance' );

WHILE v_balance > 1
LOOP
v_month_counter := v_month_counter + 1;
v_interest := ((v_new_interest_rate/100) * v_balance) / 12;
v_balance := v_balance - v_monthly_payment + v_interest;
    
DBMS_OUTPUT.PUT_LINE(v_month_counter || '      ' || TO_CHAR(v_interest,'999.99') || '    ' || TO_CHAR(v_monthly_payment,'999.99') || '    ' || TO_CHAR(v_balance, '$999,999.99'));

    IF v_balance < v_monthly_payment THEN
        v_monthly_payment := v_balance;
    END IF;
   END LOOP;
 
v_years := FLOOR(v_month_counter/12);
v_months := MOD(v_month_counter,12);

DBMS_OUTPUT.PUT_LINE('It takes '|| v_years || ' years and '|| v_months || ' month(s) to pay this loan');

EXCEPTION
WHEN OTHERS THEN
DBMS_OUTPUT.PUT_LINE('SQL Error Code' || SQLCODE);
DBMS_OUTPUT.PUT_LINE('SQL Error Code' || SQLERRM);


END;
