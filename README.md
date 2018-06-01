# Grades-

```SQL
-------------------Assignment Count by Teacher
select
sch.abbreviation as School
,crs.course_name as Course
,sec.section_number as Section
,t.lastfirst as Teacher
,count(distinct sbq.assignmentid) as Unique_Assignment_Count
,to_char(max (sbq.duedate)) as Most_Recent_Date


FROM sections sec
JOIN courses crs on sec.course_number = crs.course_number
JOIN schools sch on sch.school_number = sec.schoolid
JOIN teachers t on t.id = sec.teacher

---sbq pulls all assignments; the left join allows sections with zero assns to still show up in the output
LEFT JOIN
  (select 
  asn.duedate
  ,asn.assignmentid
  ,asn.sectionsdcid
  FROM assignmentsection asn
  WHERE asn.whocreated not in ('Legagneur, Cindy', 'Moore, Morgan','Raymond, Elizabeth') ) sbq
on sec.dcid = sbq.sectionsdcid


where sch.school_number in (1000,1001,1002,1011,1014,2000)
and sec.termid >=2700 -- limits to 17-18 sections
and t.last_name != 'Admin'  -- removes 17-18 sections with no scheduled students
group by crs.course_name, t.lastfirst, sch.abbreviation, sec.section_number
order by sch.abbreviation, crs.course_name, t.lastfirst

----------------------Assignment Weights, including school, teacher, course, section
select
sch.abbreviation as School
,crs.course_name as Course
,sec.section_number as Section
,teach.lastfirst as Teacher
,asec.assignmentid AS AssignmentID
,asec.name AS AssignmentName
,asec.weight AS Weight

from sections sec
JOIN courses crs on sec.course_number = crs.course_number
JOIN schools sch on sch.school_number = sec.schoolid
JOIN teachers teach on teach.id = sec.teacher
JOIN assignmentsection asec on asec.sectionsdcid = sec.dcid

where sch.school_number in (1000,1001,1002,1011,1014,2000)
and sec.termid >=2700 -- limits to 17-18 sections
and teach.last_name != 'Admin'  -- removes 17-18 sections with no scheduled students
and teach.last_name != 'Legagneur'

order by sch.abbreviation, teach.lastfirst
----------------------Grade Comments, including school, teacher, course, section
select
sch.abbreviation as School,
st.lastfirst AS StudentName,
cc.ID AS StudentID,
t.lastfirst as Teacher,
crs.course_name as Course,
pg.finalgradename AS FinalGradeName,
pg.grade AS FinalGrade,
sgsc.commentvalue as CommentValue,
sgsc.yearid

from cc
JOIN schools sch on sch.school_number = cc.schoolid
JOIN courses crs on crs.course_number = cc.course_number
JOIN teachers t on t.id = cc.teacherid
JOIN standardgradesectioncomment sgsc on sgsc.studentsdcid = cc.dcid
JOIN students st on st.student_number = cc.id
JOIN pgfinalgrades pg on pg.id = cc.id

where sch.school_number in (1000,1001,1002,1011,1014,2000)
--and sgsc.yearid >=19-- limits to 17-18 sections
and t.last_name != 'Admin'  -- removes 17-18 sections with no scheduled students
and crs.course_name != 'Attendance'

----------------------Historical Grades by Student
select 
c.schoolid, 
cs.course_name, 
c.termid,
s.student_number, 
s.lastfirst,
pg.finalgradename,
pg.grade,
--pg.percent,
pg.startdate,
pg.enddate
from powerschool.powerschool_cc c
JOIN powerschool.powerschool_students s on s.id = c.studentid
JOIN powerschool.powerschool_pgfinalgrades pg on pg.sectionid = c.sectionid AND pg.studentid = c.studentid
JOIN powerschool.powerschool_courses cs on cs.course_number = c.course_number
where pg.startdate > '2013-07-01' and pg.enddate > '2014-08-01'
and c.schoolid in (1000, 1001, 1002, 1011, 1014)
order by c.schoolid, cs.course_name

-----------------Assignment Data with Categories 
select 
asn.duedate
,asn.name
,asn.whocreated
,asn.weight
,asn.totalpointvalue
,sec.grade_level
,crs.credittype
,t.abbreviation
,sch.abbreviation
,crs.course_name
,d.name as "category name"
--,sec.expression
--,asn.assignmentsectionid
--,sec.termid
FROM assignmentsection asn
JOIN sections sec on sec.dcid = asn.sectionsdcid
JOIN assignmentcategoryassoc ac on ac.assignmentsectionid = asn.assignmentsectionid
JOIN teachercategory tc on tc.teachercategoryid = ac.teachercategoryid
JOIN districtteachercategory d on d.districtteachercategoryid = tc.districtteachercategoryid
JOIN schools sch on sch.school_number = sec.schoolid
JOIN courses crs on crs.course_number = sec.course_number
JOIN terms t on (asn.duedate BETWEEN t.firstday and t.lastday) and (t.schoolid = sec.schoolid)
WHERE asn.whocreated not in ('Legagneur, Cindy', 'Moore, Morgan','Raymond, Elizabeth')
and sch.school_number in (1000,1001,1002,1011,1014,2000)
and sec.termid >=2700 -- limits to 17-18 sections
--d asn.assignmentsectionid = '85752'
and t.abbreviation IN ('R1','R2', 'R3')
order by sch.abbreviation, crs.credittype, asn.whocreated; 

``
