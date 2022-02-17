0.用雪花模型生成相机ID
create schema shard_1;
create sequence shard_1.global_id_sequence;

CREATE OR REPLACE FUNCTION shard_1.id_generator(OUT result bigint) AS $$
DECLARE
    our_epoch bigint := 1314220021721;
    seq_id bigint;
    now_millis bigint;
    -- the id of this DB shard, must be set for each
    -- schema shard you have - you could pass this as a parameter too
    shard_id int := 1;
BEGIN
    SELECT nextval('shard_1.global_id_sequence') % 1024 INTO seq_id;

    SELECT FLOOR(EXTRACT(EPOCH FROM clock_timestamp()) * 1000) INTO now_millis;
    result := (now_millis - our_epoch) << 23;
    result := result | (shard_id << 10);
    result := result | (seq_id);
END;
$$ LANGUAGE PLPGSQL;

select shard_1.id_generator();


1.构造相机
do $$
declare
  v_idx integer := 0;
begin
  while v_idx < 2000 loop
    INSERT INTO huixun.camera_info(id, camera_id, camera_core_id, deleted) values (v_idx+ 18,shard_1.id_generator(),('camera_core_id_'||to_char(v_idx, 'fm00000')), 0);
    v_idx = v_idx + 1;
  end loop;
end $$;

2.构造区域
do $$
declare
  v_idx integer := 1;
begin
  while v_idx < 100 loop
    INSERT INTO huixun.huixun_area(area_id) values (shard_1.id_generator());
    v_idx = v_idx + 1;
  end loop;
end $$;


DO $$ DECLARE
v_idx INTEGER := 0;
DECLARE
	i INT;
DECLARE
	cnt INT;
BEGIN
		cnt := 0;
		
	SELECT COUNT
		( 1 ) INTO cnt 
	FROM
		huixun_area;
	while
	v_idx < cnt
	loop
	i := 0;
	INSERT INTO huixun_area_camera_mapping_copy1 (
		SELECT
			ha.area_id,
			ci.camera_id,
			'camera_core_id',
			0 
		FROM
			huixun_area ha,
			camera_info ci 
			LIMIT  trunc( random() * ( 20-1 ) + 1 )
			offset v_idx * 20
		);
	v_idx := v_idx + 1;
	
END loop;

END $$;

3.构造区域场景关系
DO $$ DECLARE
v_idx INTEGER := 0;
DECLARE
	i INT;
DECLARE
	cnt INT;
BEGIN
		cnt := 0;
		
	SELECT COUNT
		( 1 ) INTO cnt 
	FROM
		huixun.huixun_area;
	while
	v_idx < cnt
	loop
	i := 0;
	INSERT INTO huixun.huixun_area_scene_mapping (
		SELECT
			ha.area_id,
			(array[10101,10301,10501,10502,10601,10701,10901,11101,19999])[floor(random()*9)::int + 1],
			0 
		FROM
			huixun.huixun_area ha limit 1 offset v_idx
		);
	v_idx := v_idx + 1;
	
END loop;

END $$;





4.相机场所
DO $$ DECLARE
v_idx INTEGER := 0;
DECLARE
	i INT;
DECLARE
	cnt INT;
BEGIN
		cnt := 0;
		
	SELECT COUNT
		( 1 ) INTO cnt 
	FROM
		huixun.camera_info;
	while
	v_idx < cnt
	loop
	i := 0;
	INSERT INTO huixun.huixun_camera_location_mapping (
		SELECT
			ha.camera_id,
			(array[10101,10301,10501,10502,10601,10701,10901,11101,19999])[floor(random()*9)::int + 1]
		
		FROM
			huixun.camera_info ha limit 1 offset v_idx
		);
	v_idx := v_idx + 1;
	
END loop;

END $$;





## 第二个
create table huixun.profile as SELECT profile_id from profile limit 2000000;
create table huixun.camera_bak as SELECT camera_core_id from huixun.camera_info WHERE camera_id in (1031, 1032,1043, 1044, 1045)



INSERT INTO huixun.test_profile_count_hour_camera ( captured_hour, profile_id, camera_core_id, face_count ) SELECT
captured_hour,
profile_id,
camera_core_id,
1 
FROM
	( SELECT * FROM huixun.test_profile ) M,
	huixun.test_camera,
	huixun.test_time_hour



DO $$ DECLARE
i INTEGER := 0;
BEGIN
		while
		i < 90  loop
		INSERT INTO test_date ( captured_date )
	VALUES
		(
			TO_TIMESTAMP('2019-10-07 16:00:00', 'YYYY-MM-DD hh24:mi:ss') + make_interval(days => i)
		);
	i = i + 1;
	
END loop;

END $$;



## 第三个

insert into huixun.huixun_profile_count_day (captured_date, profile_id, face_count)  select '2019-10-27 16:00:00', profile_id, 5 from profile  order by random()  limit 5000000 ON CONFLICT (captured_date, profile_id) do NOTHING



select count(distinct profile_id)  from huixun.huixun_profile_count_day where captured_date='2019-10-22 16:00:00'


select count(*) from huixun_profile_count_day 

select * from profile_first_appearance where captured_time > '2019-10-26 00:00:00'


SELECT count( DISTINCT  profile_id) FROM huixun.huixun_profile_count_day  WHERE captured_date >= to_timestamp(1571976000) AND captured_date < to_timestamp(1572408000)


select count(*) from huixun_profile_count_day


select count(1) huixun.huixun_profile_count_day (captured_date, profile_id, face_count)  where capture '2019-10-27 16:00:00', 


create table huixun.two_w_profile_id as select profile_id  from huixun.huixun_profile_count_day where captured_date='2019-10-22 16:00:00'


select * from random_camera_id

create table huixun.random_camera_id as 
select min(camera_core_id) as core_id  , area_id from camera_info c, public.camera_area_mapping m where m.camera_str = c.camera_core_id 
group by area_id  order by area_id 


insert into huixun.profile_count_hour_camera_copy1 (captured_hour, profile_id, camera_core_id,face_count)  select captured_hour, profile_id, core_id as camera_core_id, 1 from (select * from two_w_profile_id ) m, random_camera_id


## 第四个
do $$
declare
  v_idx integer := 100001;
begin
  while v_idx < 500001 loop
    INSERT INTO huixun.profile_ye_chu_one_day(captured_date,profile_id) values ('2019-06-11 16:00:00',v_idx);
    v_idx = v_idx + 1;
  end loop;
end $$;



do $$
declare
  v_idx integer := 100;
begin
  while v_idx < 10000000 loop
    INSERT INTO public.face_cluster_mapping(face_id,cluster_id) values (to_char(v_idx, '99999999'), to_char(v_idx, '99999999'));
    v_idx = v_idx + 1;
  end loop;
end $$;




insert into huixun.huixun_profile_count_day (captured_date, profile_id, face_count)  select '2019-10-27', profile_id, 10 from profile order by random()  limit 2000000 ON CONFLICT (captured_date, profile_id) do NOTHING


## 第五个
DELETE from face_cluster_mapping where face_id like '  6%'