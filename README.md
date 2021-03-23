# AWS-South-Korea-SAS-Data-Crawling
%extracResource(RESTQuery="https://www.weather.go.kr/weather/observation/aws_table_popup.jsp", 
	FSOutput="C:\Program Files\SASUniversityEdition\myfolders");

%macro doMacroTemplate(inputvariableName=, BOOLEANVAR=);
	/*ERROR HANDLING:*/
%IF &inputvariableName=%THEN
		%DO;
			%put ERROR : A null value is not vaild.;
			%put ERROR- Please provide an Variable Name.;
			%return;
		%END;

	/*FLOW CONTROL:*/

	%IF &BOOLEANVAR=TRUE %THEN
		%DO;
			%PUT "Boolean is TRUE.";
			%PUT Input Name is &InputVariableName.;
		%END;
	%ELSE
		%DO;
			%PUT "Boolean is NOT TRUE.";
			%PUT Input Name is&InputVariableName.;
		%END;
	&mend doMacroTemplate;
	%web_drop_table(WORK.IMPORT);
	*FILENAME REFFILE '/folders/myshortcuts/Myfolders/pg194_ue/EPG194/data/AWS.xlsx';

	PROC IMPORT DATAFILE=REFFILE DBMS=XLSX OUT=WORK.IMPORT;
		GETNAMES=YES;
	RUN;

	DATA WORK.IMPORT; missing;
		keep 고도 기온 풍속1(km/h) 풍향10 풍속10(km/h) 습도;
		if 고도 ne . or '' then do;
		 else if 고도 > 0 then do;
		end;
		if 기온 ne . or '' then do;
		 else if 기온 => 0 and <= 0 then do;
	    end;
		if 풍향10 ne . then do;
		 else if 풍향10 ne . or '' then do;
		end;
		if 풍속10(km/h) ne . or '' then do;
		 else if 풍속10(km/h) >= 0 then do;
		end;
		if 습도 ne . or '' then do;
		 else if 습도 >= 0 then do;
		end;
        풍속1(km/h)=mob(ceil(1000*풍속1(km/h),1));
        풍속10(km/h)=mob(ceil(1000*풍속10(km/h),1));
        format 고도 기온 풍속1(km/h) 풍향10 풍속10(km/h) 습도;
        output;
	RUN;
	
	PROC FREQ DATA=WORK.IMPORT nlevels;
		tables 고도 기온 풍속1(km/h) 풍향10 풍속10(km/h) 습도 /nocum nopercent;
	RUN;

    TITLE1 'AWS지점/고도 각 기준 별 당일 최대 풍속';
    TITLE2 '각 풍속 당 데이터 값';
	proc print data=WORK.IMPORT noobs;
	run;
	
	/*****/
	DATA TempHumid;
	SET WORK.IMPORT; 
	RUN;

	PROC SORT DATA=TempHumid;
		By 풍속10(km/h) descending;
    RUN;

	PROC PRINT DATA=TempHumid;
	Run;
