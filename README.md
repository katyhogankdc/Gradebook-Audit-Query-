# Gradebook-Audit-Query-

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
,asec.assignmentsectionid as AssignmentSectionID
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

----------------------Grade Comments, including school, teacher, course, section
select
sch.abbreviation as School,
st.lastfirst AS StudentName,
cc.ID AS StudentID,
t.lastfirst as Teacher,
crs.course_name as Course,
cc.section_number AS Section,
pg.finalgradename AS FinalGradeName,
pg.grade AS FinalGrade,
pg.percent AS PercentGrade,
sec.comment_value as CommentValue

from cc
JOIN schools sch on sch.school_number = cc.schoolid
JOIN courses crs on crs.course_number = cc.course_number
JOIN teachers t on t.id = cc.teacherid
JOIN sections sec on sec.section_number = cc.section_number
JOIN students st on st.student_number = cc.id
JOIN pgfinalgrades pg on pg.id = cc.id

where sch.school_number in (1000,1001,1002,1011,1014,2000)
and sec.termid >=2700 -- limits to 17-18 sections
and t.last_name != 'Admin'  -- removes 17-18 sections with no scheduled students

