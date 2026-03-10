# Khởi đầu:
```bash
$LABTAINER_DIR/labs
wget https://github.com/SangPK34/An-toan-he-dieu-hanh/raw/main/BTCN/lab2025.tar -O lab2025.tar && tar -xvf lab2025.tar
mv ../../labs/processnice/dockerfiles/Dockerfile.process_nice.process_nice.student ../../labs/processnice/dockerfiles/Dockerfile.processnice.process_nice.student
cd $LABTAINER_DIR/scripts/labtainer-student 
rebuild -f -b processnice

#cho chắc:
imodule https://github.com/SangPK34/An-toan-he-dieu-hanh/raw/main/BTCN/lab2025.tar
```
# Bài 1:
```bash
chmod +x ~/labtainer/trunk/labs/processnice/instr_config/*.sh
find ~/labtainer/trunk/labs/processnice -exec touch {} +
labtainer processnice
```
# Bài 3:
```bash
find ~/labtainer/trunk/labs/oss_eval_auditd_n4_tien_4 -exec touch {} +
docker rm -f $(docker ps -aq --filter name=igrader) 2>/dev/null
find ~/labtainer/trunk/labs/oss_eval_auditd_n4_tien_4/instr_config -type f -exec sed -i 's/\r$//' {} +
chmod -R +x ~/labtainer/trunk/labs/oss_eval_auditd_n4_tien_4/instr_config
docker rmi oss_eval_auditd_n4_tien_4.ubuntu.student.grader 2>/dev/null
```
# Bài 4:
```bash
find ~/labtainer/trunk/labs/oss_eval_clamav_n4_minh_3 -exec touch {} +
docker rm -f $(docker ps -aq --filter name=igrader) 2>/dev/null
find ~/labtainer/trunk/labs/oss_eval_clamav_n4_minh_3/instr_config -type f -exec sed -i 's/\r$//' {} +
chmod -R +x ~/labtainer/trunk/labs/oss_eval_clamav_n4_minh_3/instr_config
```
# Bài 5:
```bash
find ~/labtainer/trunk/labs/cmd_log -exec touch {} +
docker rm -f $(docker ps -aq --filter name=igrader) 2>/dev/null
find ~/labtainer/trunk/labs/cmd_log/instr_config -type f -exec sed -i 's/\r$//' {} +
chmod -R +x ~/labtainer/trunk/labs/cmd_log/instr_config
docker rmi cmd_log.ubuntu.student.grader 2>/dev/null
```
