```bash
$LABTAINER_DIR/labs
wget https://github.com/SangPK34/An-toan-he-dieu-hanh/raw/main/BTCN/lab2025.tar -O lab2025.tar && tar -xvf lab2025.tar
mv ../../labs/processnice/dockerfiles/Dockerfile.process_nice.process_nice.student ../../labs/processnice/dockerfiles/Dockerfile.processnice.process_nice.student
cd $LABTAINER_DIR/scripts/labtainer-student 
rebuild -f -b processnice

#cho chắc:
imodule https://github.com/SangPK34/An-toan-he-dieu-hanh/raw/main/lab2025.tar
```
Bài1:
```bash
chmod +x ~/labtainer/trunk/labs/processnice/instr_config/*.sh
find ~/labtainer/trunk/labs/processnice -exec touch {} +
labtainer processnice
```
