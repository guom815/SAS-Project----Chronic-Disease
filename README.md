/*Part I a*/
libname analysis "/home/u63582559/BS723 Project 2";
/*Part I b*/
data transplant2;
	set analysis.proj2_data;
	/*Part I bi*/
	if tx_received=1;
	/*Part I bii*/
	if diag=3 then diagnew=1;
	else if diag in (1,2,4,5,6) then diagnew=2;
	else if diag=7 then diagnew=3;
	/*Part I biii*/
	if diagnew=3 then delete;
run;
/*Part I c*/
proc import out=donor
	datafile="/home/u63582559/BS723 Project 2/donor_2023.csv"
	DBMS=csv replace;
	getnames=yes;
run;	
/*Part I d&e*/
proc sort data=donor;
	by PT_CODE;
run;
proc sort data=transplant2;
	by PT_CODE;
run;
data alltransplant;
  	merge donor (in=d) transplant2 (in=t);
 	by PT_CODE;
 	if d and not t then delete;
run;
/*Part I f*/
data alltransplant_new;
	set alltransplant;
	if (SEX_DON="F" and female=1) or (SEX_DON="M" and female=0) then sex_mismatch=0;
	else if (SEX_DON="M" and female=1)  or (SEX_DON="F" and female=0) then sex_mismatch=1;
run;
proc print data=alltransplant_new;
run;

/*Part II ai*/
proc sort data=alltransplant_new out=alltransplant_new_1;
	by diagnew;
run;
proc boxplot data=alltransplant_new_1;
	plot gfr_change*diagnew / cboxes=black;
run;
proc ttest data=alltransplant_new_1;
	class diagnew; 
	var gfr_change; 
run;
/*Part II aiii*/
proc sort data=alltransplant_new; 
	by descending diabetes descending diagnew; 
run;
proc freq data=alltransplant_new order=data;
	tables diabetes*diagnew / chisq measures;
run;
/*Part II b*/
proc reg data=alltransplant_new;
  	model gfr_change = diabetes diagnew age_list bmi_list ethcat4 female;
run;
proc glm data=alltransplant_new;
	class diabetes (ref="0") diagnew (ref="2") ethcat4 (ref="1") female (ref="0");
	model gfr_change = diabetes diagnew age_list bmi_list ethcat4 female / solution;	
	means diabetes;
	lsmeans diabetes/stderr; 
run;

/*Part III a*/
proc logistic data=alltransplant_new descending;
	class diabetes (ref="0") diagnew (ref="2") ethcat4 (ref="1") female (ref="0") sex_mismatch (ref="0");
	model pstatusdc=listingvol diabetes diagnew age_list bmi_list ethcat4 female sex_mismatch age_don;
run;
/*Part III b*/
proc glm data=alltransplant_new;
	class diabetes (ref="0") diagnew (ref="2") ethcat4 (ref="1") sex_mismatch (ref="0");
	model pstatusdc=sex_mismatch|diabetes diagnew age_list age_don ethcat4 / solution;
run;

		
<img width="468" height="635" alt="image" src="https://github.com/user-attachments/assets/656e79c2-abea-4e81-b458-d04ae31bc7e8" />
