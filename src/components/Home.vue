<template>
  <div>
    <el-header>
      <div class="head">广播星历转换GNSS卫星坐标</div>
    </el-header>
    <el-main>
      <el-card>
        <p>如何使用：</p>
        <div>
          1.
          点击导入文件，选择需要进行分析的文件，确定后出现等待数据导入即文件数据读取完毕
        </div>
        <div>
          2.
          点击数据导入，这一步可能需要耐心等待一段时间，成功后列表将会展示数据
        </div>
        <div>3. 每一行都有计算位置的功能，点击即可进行卫星位置计算</div>
      </el-card>
      <el-row :gutter="20" style="margin-top: 20px">
        <el-button
          type="primary"
          id="fileImport"
          @click="clickLoad"
          size="mini"
          round
          style="margin: 10px"
          >导入文件</el-button
        >
        <input
          type="file"
          id="files"
          ref="refFile"
          style="display: none"
          @change="fileLoad"
        />
        <el-button
          id="python"
          @click="importData"
          size="mini"
          round
          style="margin: 10px"
          >数据导入</el-button
        >
      </el-row>
      <el-card style="margin-top: 20px">
        <el-table
          :data="n_dic_list"
          style="width: 100%"
          stripe
          height="400"
          v-loading="loading"
          element-loading-text="等待数据导入"
        >
          <el-table-column prop="r" label="数据组数" width="80">
          </el-table-column>
          <el-table-column prop="PRN" label="卫星PRN号" width="100">
          </el-table-column>
          <el-table-column prop="TIME" label="历元" width="180">
          </el-table-column>
          <el-table-column prop="a0" label="卫星钟偏差(s)" width="180">
          </el-table-column>
          <el-table-column prop="a1" label="卫星钟漂移(s/s)" width="180">
          </el-table-column>
          <el-table-column prop="a2" label="卫星钟漂移速度(s/s*s)" width="80">
          </el-table-column>
          <el-table-column prop="IODE" label="IODE" width="80">
          </el-table-column>
          <el-table-column prop="Crs" label="Crs" width="120">
          </el-table-column>
          <el-table-column prop="Deln" label="Deln" width="180">
          </el-table-column>
          <el-table-column prop="M0" label="M0" width="180"> </el-table-column>
          <el-table-column prop="Cuc" label="Cuc" width="180">
          </el-table-column>
          <el-table-column prop="e" label="e" width="180"> </el-table-column>
          <el-table-column prop="Cus" label="Cus" width="180">
          </el-table-column>
          <el-table-column prop="sqrt_A" label="sqrt_A" width="180">
          </el-table-column>
          <el-table-column prop="TOE" label="TOE" width="100">
          </el-table-column>
          <el-table-column prop="Cic" label="Cic" width="180">
          </el-table-column>
          <el-table-column prop="OMEGA" label="OMEGA" width="180">
          </el-table-column>
          <el-table-column prop="Cis" label="Cis" width="180">
          </el-table-column>
          <el-table-column prop="i0" label="i0" width="180"> </el-table-column>
          <el-table-column prop="Crc" label="Crc" width="100">
          </el-table-column>
          <el-table-column prop="omega" label="omega" width="180">
          </el-table-column>
          <el-table-column prop="OMEGA_DOT" label="OMEGA DOT" width="180">
          </el-table-column>
          <el-table-column prop="IDOT" label="IDOT" width="180">
          </el-table-column>
          <el-table-column prop="L2_code" label="L2_code" width="80">
          </el-table-column>
          <el-table-column prop="GPSweek" label="GPSweek" width="80">
          </el-table-column>
          <el-table-column prop="L2_P_code" label="L2_P_code" width="80">
          </el-table-column>
          <el-table-column prop="卫星精度(m)" label="卫星精度(m)" width="80">
          </el-table-column>
          <el-table-column prop="卫星健康状态" label="卫星健康状态" width="80">
          </el-table-column>
          <el-table-column prop="TGD(s)" label="TGD(s)" width="180">
          </el-table-column>
          <el-table-column prop="IODC" label="IODC" width="80">
          </el-table-column>
          <el-table-column fixed="right" label="操作" width="100">
            <template slot="header">
              <el-button size="mini" type="primary" round @click="exportCsv"
                >导出列表</el-button
              >
            </template>
            <template slot-scope="scope">
              <el-button size="mini" @click="calculate(scope.$index, scope.row)"
                >计算位置</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </el-card>
      <el-card style="margin-top: 20px">
        <el-descriptions title="卫星位置" border size="mini" :column="1">
          <el-descriptions-item label="历元"
            >{{ cal.year }}年{{ cal.month }}月{{ cal.day }}日{{ cal.hour }}时{{
              cal.minute
            }}分{{ cal.second }}秒</el-descriptions-item
          >
          <el-descriptions-item label="卫星PRN号">{{
            cal.PRN
          }}</el-descriptions-item>
          <el-descriptions-item label="平均角速度">{{
            cal.n
          }}</el-descriptions-item>
          <el-descriptions-item label="平近点角">{{
            cal.Mk
          }}</el-descriptions-item>
          <el-descriptions-item label="偏近点角">{{
            cal.E
          }}</el-descriptions-item>
          <el-descriptions-item label="真近点角">{{
            cal.Vk
          }}</el-descriptions-item>
          <el-descriptions-item label="升交距角">{{
            cal.u0
          }}</el-descriptions-item>
          <el-descriptions-item label="摄动改正项"
            >{{ cal.Delu }}, {{ cal.Delr }},
            {{ cal.Deli }}</el-descriptions-item
          >
          <el-descriptions-item label="摄动改正后的升交距角、卫星矢径和轨道倾角"
            >{{ cal.u }}, {{ cal.r }}, {{ cal.i }}</el-descriptions-item
          >
          <el-descriptions-item label="轨道平面坐标X,Y"
            >{{ cal.xk }}, {{ cal.yk }}</el-descriptions-item
          >
          <el-descriptions-item label="观测时刻升交点经度">{{
            cal.OMEGAk
          }}</el-descriptions-item>
          <el-descriptions-item label="地固直角坐标系X, Y, Z"
            >{{ cal.Xk }}, {{ cal.Yk }}, {{ cal.Zk }}</el-descriptions-item
          >
        </el-descriptions>
      </el-card>
    </el-main>
  </div>
</template>

<script>

import Papa from "papaparse";
export default {
  name: "Home",
  data() {
    return {
      fileName: "",
      size: "",
      fileInfo: "",
      dataInfo: [],
      n_dic_list: [],
      loading: false,
      cal: {},
    };
  },
  methods: {
    clickLoad() {
      this.$refs.refFile.dispatchEvent(new MouseEvent("click"));
    },
    fileLoad() {
      const selectedFile = this.$refs.refFile.files[0];
      this.fileName = selectedFile.name;
      // this.size = selectedFile.size;
      console.log("文件名：" + this.fileName + "大小：" + this.size);
      let reader = new FileReader();
      reader.readAsText(selectedFile);
      reader.onload = () => {
        // console.log(reader.result);
        this.fileInfo = reader.result;
        // console.log(this.fileInfo);
      };
      this.loading = true;
      this.n_dic_list = [];
      this.$message({ message: "数据读取成功", type: "success" });
    },
    importData() {
      // console.log(this.fileInfo);
      this.dataInfo = this.fileInfo.split(/[(\r\n)\r\n]+/);
      // console.log(this.dataInfo);
      const startNum = this.start();
      const totalDataLine = parseInt((this.dataInfo.length - startNum) / 8);
      console.log("共" + totalDataLine + "组数据");
      for (let j = 0; j < totalDataLine; j++) {
        let n_dic = {};
        for (let i = 0; i < 8; i++) {
          const dataContent = this.dataInfo[startNum + 8 * j + i];
          // console.log(dataContent);
          n_dic["r"] = j + 1;
          if (i == 0) {
            n_dic["PRN"] = parseInt(dataContent.substring(0, 2).trim());
            n_dic["TIME"] = dataContent.substring(3, 22);
            n_dic["a0"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["a1"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["a2"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
          if (i == 1) {
            n_dic["IODE"] = parseFloat(
              (
                dataContent.substring(3, 18) +
                "e" +
                dataContent.substring(19, 22)
              ).trim()
            );
            n_dic["Crs"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["Deln"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["M0"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
          if (i == 2) {
            n_dic["Cuc"] = parseFloat(
              (
                dataContent.substring(3, 18) +
                "e" +
                dataContent.substring(19, 22)
              ).trim()
            );
            n_dic["e"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["Cus"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["sqrt_A"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
          if (i == 3) {
            n_dic["TOE"] = parseFloat(
              (
                dataContent.substring(3, 18) +
                "e" +
                dataContent.substring(19, 22)
              ).trim()
            );
            n_dic["Cic"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["OMEGA"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["Cis"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
          if (i == 4) {
            n_dic["i0"] = parseFloat(
              (
                dataContent.substring(3, 18) +
                "e" +
                dataContent.substring(19, 22)
              ).trim()
            );
            n_dic["Crc"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["omega"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["OMEGA_DOT"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
          if (i == 5) {
            n_dic["IDOT"] = parseFloat(
              (
                dataContent.substring(3, 18) +
                "e" +
                dataContent.substring(19, 22)
              ).trim()
            );
            n_dic["L2_code"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["GPSweek"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["L2_P_code"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
          if (i == 6) {
            n_dic["卫星精度(m)"] = parseFloat(
              (
                dataContent.substring(3, 18) +
                "e" +
                dataContent.substring(19, 22)
              ).trim()
            );
            n_dic["卫星健康状态"] = parseFloat(
              (
                dataContent.substring(22, 37) +
                "e" +
                dataContent.substring(38, 41)
              ).trim()
            );
            n_dic["TGD(s)"] = parseFloat(
              (
                dataContent.substring(41, 56) +
                "e" +
                dataContent.substring(57, 60)
              ).trim()
            );
            n_dic["IODC"] = parseFloat(
              (
                dataContent.substring(60, 75) +
                "e" +
                dataContent.substring(76, 79)
              ).trim()
            );
          }
        }
        this.n_dic_list.push(n_dic);
      }
      this.total = this.n_dic_list.length;
      this.loading = false;
      this.$message({ message: "数据导入成功", type: "success" });
    },
    // 起始行设置
    start() {
      let startNum = 0;
      for (let i = 0; i < this.dataInfo.length; i++) {
        if (this.dataInfo[i].includes("END OF HEADER")) {
          startNum = i + 1;
        }
      }
      console.log(startNum);
      return startNum;
    },
    // 卫星位置计算
    calculate(index, row, t1) {
      let cal = {};
      console.log(index, row);
      //计算卫星运行平均角速度 GM：WGS84下引力常数 a：长半径
      const GM = 398600500000000;
      const n0 = Math.sqrt(GM) / Math.pow(row.sqrt_A, 3);
      const n = n0 + row.Deln;
      //计算归化时间tk 计算t时刻卫星位置 UT：世界时(h)
      let year = parseInt(row.TIME.substring(0, 3).trim());
      let month = parseInt(row.TIME.substring(3, 6).trim());
      let day = parseInt(row.TIME.substring(6, 9).trim());
      let hour = parseInt(row.TIME.substring(9, 12).trim());
      let minute = parseInt(row.TIME.substring(12, 15).trim());
      let second = parseFloat(row.TIME.substring(15, 20).trim());
      const UT = hour + minute / 60 + second / 3600;
      //GPS起始时刻1980年1月6日0时
      if (year >= 80) {
        if (year == 80 && month == 1 && day < 6) {
          year += 2000;
        } else {
          year += 1900;
        }
      } else {
        year += 2000;
      }
      if (month <= 2) {
        year -= 1;
        month += 12;
      }
      //将当前计算的时刻线转换为儒略日再转换为GPS时间
      let JD =
        365.25 * year +
        parseInt(30.6001 * (month + 1)) +
        day +
        UT / 24 +
        1720981.5;
      let WN = parseInt((JD - 2444244.5) / 7); //GPS_week num 目标时刻GPS周
      let toc = (JD - 2444244.5 - 7.0 * WN) * 24 * 3600.0; //目标时刻GPS秒
      let tk = 0;
      //对观测时刻t1进行钟差改正
      if (t1 == null) {
        tk = 0;
      } else {
        let Delt = (row.a0 + row.a1 * (t1 - toc) + row.a2 * (t1 - toc)) ^ 2;
        let t = t1 - Delt;
        tk = t - row.TOE;
      }
      //平近点角计算Mk
      let Mk = row.M0 + n * tk;
      //偏近点角计算Ek
      let E = 0;
      let E1 = 1;
      let count = 0;
      while (Math.abs(E1 - E) > 1e-10) {
        count += 1;
        E1 = E;
        E = Mk + row.e * Math.sin(E);
        if (count > 1e8) {
          this.$message({ message: "计算偏近点角时未收敛", type: "warning" });
          break;
        }
      }
      //真近点角计算
      let Vk = Math.atan(
        (Math.sqrt(1 - row.e * row.e) * Math.sin(E)) / (Math.cos(E) - row.e)
      );
      //升交距角计算
      let u0 = Vk + row.omega;
      //摄动改正项Delu、Delr、Deli：升交距角u、卫星矢径r和轨道倾角i的摄动量
      let Delu = row.Cuc * Math.cos(2 * u0) + row.Cus * Math.sin(2 * u0);
      let Delr = row.Crc * Math.cos(2 * u0) + row.Crs * Math.sin(2 * u0);
      let Deli = row.Cic * Math.cos(2 * u0) + row.Cis * Math.sin(2 * u0);
      //计算经过摄动改正的升交距角uk、卫星矢径rk和轨道倾角ik
      let u = u0 + Delu;
      let r = Math.pow(row.sqrt_A, 2) * (1 - row.e * Math.cos(E)) + Delr;
      let i = row.i0 + Deli + row.IDOT * tk;
      //计算卫星在轨道平面坐标系的坐标，卫星在轨道平面直角坐标系（x轴指向升交点）中的坐标
      let xk = row.r * Math.cos(u);
      let yk = row.r * Math.sin(u);
      //观测时刻升交点经度OMEGAk的计算，升交点经度OMEGAk等于观测时刻升交点赤经OMEGA与格林尼治恒星时GAST之差
      let omegae = 7.292115e-5;
      let OMEGAk = row.OMEGA + (row.OMEGA_DOT - omegae) * tk - omegae * row.TOE;
      //计算卫星在地固系中的直角坐标
      let Xk = xk * Math.cos(OMEGAk) - yk * Math.cos(i) * Math.sin(OMEGAk);
      let Yk = xk * Math.sin(OMEGAk) + yk * Math.cos(i) * Math.cos(OMEGAk);
      let Zk = yk * Math.sin(i);
      //恢复历元
      if (month > 12) {
        year += 1;
        month -= 12;
      }
      cal.year = year;
      cal.month = month;
      cal.day = day;
      cal.hour = hour;
      cal.minute = minute;
      cal.second = second;
      cal.PRN = row.PRN;
      cal.n = n;
      cal.Mk = Mk;
      cal.E = E;
      cal.Vk = Vk;
      cal.u0 = u0;
      cal.Delu = Delu;
      cal.Delr = Delr;
      cal.Deli = Deli;
      cal.u = u;
      cal.r = r;
      cal.i = i;
      cal.xk = xk;
      cal.yk = yk;
      cal.OMEGAk = OMEGAk;
      cal.Xk = Xk;
      cal.Yk = Yk;
      cal.Zk = Zk;

      console.log(cal);
      this.cal = cal;
    },
    //导出列表
    exportCsv() {
      if (!this.fileName) {
        this.$message({ message: "请导入文件数据", type: "warning" });
      } else {
        const csv = Papa.unparse(this.n_dic_list);
        let content = new Blob([csv]);
        let urlObject = window.URL || window.webkitURL || window;
        let url = urlObject.createObjectURL(content);
        let el = document.createElement("a");
        el.href = url;
        el.download = this.fileName + ".csv";
        el.click();
        urlObject.revokeObjectURL(url);
      }
    },
  },
};
</script>

<style scoped>
.head {
  font-size: 40px;
}
</style>
