<template>
  <div @click="(dialogFormVisible = true), getCards()">
    <div class="content-tag">
      <Tag v-for="(label, index) in labels" :key="index" :label="label" />
    </div>
    <div class="content-text" type="text">
      <p>
        {{ card.title }}
      </p>
    </div>
    <div class="content-end">
      <div v-show="formatDate2(cardData.deadline) !== 'Invalid date'">
        <div
          v-if="cardData.status === 1"
          class="content-end-detail"
          style="background-color: #61bd4f; color: white"
        >
          <i class="el-icon-time"></i> {{ formatDate2(cardData.deadline) }}
        </div>
        <div
          v-else
          class="content-end-detail"
          style="background-color: #ec9488; color: white"
        >
          <i class="el-icon-time"></i> {{ formatDate2(cardData.deadline) }}
        </div>
      </div>
      <div v-show="card.description" class="content-end-detail">
        <i class="el-icon-tickets"></i>
      </div>
      <div v-show="fileLists.length" class="content-end-detail"><i class="el-icon-files"></i>{{fileLists.length}}</div>
      <div v-show="checkLists.length" class="content-end-detail">
        <i class="el-icon-document-checked"></i>
        <!-- 2/{{ checkLists.length }} -->
      </div>
    </div>

    <!-- edit detail -->
    <el-dialog
      width="750px"
      :append-to-body="true"
      :visible.sync="dialogFormVisible"
    >
      <el-row>
        <el-col :span="1">
          <div class="left-title">
            <i class="el-icon-bank-card"></i>
          </div>
        </el-col>
        <el-col :span="23">
          <div class="right-title">
            <p v-if="titleClickCheck" @click="clickTitleNode()">
              {{ cardData.title }}
            </p>
            <el-input
              v-else
              type="textarea"
              @focusout.native="outTitleNode()"
              @keyup.enter.native="outTitleNode()"
              autosize
              autofocus
              v-model="dataTitle"
            ></el-input>
            <!-- <span>Trong danh sách <b>{{ card.title }}</b></span> -->
          </div>
        </el-col>
      </el-row>
      <el-row :gutter="15" class="dialog-container">
        <el-col :span="17">
          <el-row v-show="labels.length">
            <el-col :span="2" style="font-size: 1px"> . </el-col>
            <el-col :span="22">
              <div class="tag-detail">
                <p class="title-tag-detail">NHÃN</p>
                <div class="tag-detail-tag">
                  <div
                    v-for="(label) in labels"
                    :key="label.id"
                    :style="{ background: label.color }"
                  >
                    {{ label.name }}
                  </div>
                </div>
              </div>
            </el-col>
          </el-row>
          <el-row v-show="cardData.deadline" class="magin-mission">
            <el-col :span="2" style="font-size: 1px"> . </el-col>
            <el-col :span="22">
              <div class="tag-detail">
                <p class="title-tag-detail">NGÀY HẾT HẠN</p>
                <div class="checkbox-date">
                  <input
                    type="checkbox"
                    v-if="cardData.status == 1"
                    checked
                    @click="checkBoxDateDate"
                  />
                  <input type="checkbox" v-else @click="checkBoxDateDate" />
                  <div>
                    {{ cardData.deadline }}
                    <span
                      v-if="cardData.status == 1"
                      style="background-color: #61bd4f;"
                      >HOÀN TẤT</span
                    >
                    <span
                      v-if="
                        cardData.deadline <= formatDate(new Date()) &&
                        cardData.status == 0
                      "
                      style="background-color: #ec9488;"
                      >HẾT HẠN</span
                    >
                    <span
                      v-else-if="
                        cardData.deadline <= formatDate(new Date().setDate(new Date().getDate() + 1)) &&
                        cardData.status == 0
                      "
                      style="background-color: #f2d600;"
                      >GẦN HẾT HẠN</span
                    >
                  </div>
                </div>
              </div>
            </el-col>
          </el-row>
          <el-row class="magin-mission">
            <el-col :span="2" style="font-size: 24px"
              ><i class="el-icon-document"></i
            ></el-col>
            <el-col :span="22">
              <div class="note-card">
                <p class="title-tag-detail">MÔ TẢ</p>
                <p
                  v-if="titleClickCard && card.description"
                  @click="clickTitleCard"
                >
                  {{ card.description }}
                </p>
                <el-input
                  v-else
                  type="textarea"
                  placeholder="Thêm mô tả chi tiết hơn..."
                  @focusout.native="outTitleCard"
                  @keyup.enter.native="outTitleCard"
                  v-model="dataTitleCard"
                ></el-input>
              </div>
            </el-col>
          </el-row>
          <el-row v-show="fileLists.length" class="magin-mission">
            <el-col :span="2" style="font-size: 24px"
              ><i class="el-icon-files"></i
            ></el-col>
            <el-col :span="22">
              <div class="note-card">
                <p class="title-tag-detail">TẬP TIN ĐÍNH KÈM</p>
                <el-row :gutter="20">
                  <el-col v-for="fileList in fileLists" :key="fileList.id" :span="12">
                    <div>
                      <el-image
                        :src="'http://vuecourse.zent.edu.vn/storage/'+fileList.path"
                      ></el-image>
                      <el-input v-if="checkNameFile===fileList.id" @keyup.enter.native="editFile(fileList.id)" size="mini" placeholder="Nhập tên tệp..." v-model="dataNameFile"></el-input>
                      <p v-else class="file-name">{{fileList.name}}</p>
                    </div>
                    <div class="button-file-edit">
                      <el-button v-if="checkNameFile===fileList.id" type="text" @click="editFile(fileList.id)"
                        >Lưu</el-button
                      > 
                      <el-button v-else type="text" @click="checkNameFile=fileList.id,dataNameFile=fileList.name"
                      >Sửa</el-button
                    > 
                      <el-button v-if="checkNameFile===fileList.id" type="text" @click="checkNameFile='',dataNameFile=''"
                        >Thoát</el-button
                      > 
                      <el-button v-else type="text" @click="deleteFile(fileList.id)"
                      >Loại bỏ</el-button
                    >
                    </div>
                  </el-col>
                </el-row>
              </div>
            </el-col>
          </el-row>
          <el-row
            v-for="checkL,index in checkLists"
            :key="index"
            class="magin-mission"
          >
            <CheckListsComponent :index="index" :card="card" :checkL="checkL" :getCards="getCards" :getdirectories="getdirectories"/>
          </el-row>
        </el-col>
        <el-col :span="7">
          <div class="menu-detail">
            <p class="title-tag-detail">THÊM VÀO THẺ</p>
            <div class="title-tag-detail-button">
              <el-popover
                class="labels"
                placement="bottom"
                width="300"
                trigger="click"
              >
                <div v-if="checkAddCard">
                  <el-input placeholder="Tìm nhãn..." v-model="sheachCards" style="margin-bottom: 10px;"></el-input>
                  <p class="title-tag-detail">NHÃN</p>
                  <div>
                    <el-row
                      v-for="(labelsList, index) in labelsLists"
                      :key="index"
                      :gutter="10"
                    >
                      <el-col
                        @click.native="attachLabels(labelsList.id)"
                        :span="21"
                        ><div
                          class="tag-content"
                          :style="{ background: labelsList.color }"
                        >
                          <span>{{ labelsList.name }}</span>
                          <i
                            v-for="(label, index) in labels"
                            :key="index"
                            v-show="labelsList.id == label.id"
                            class="el-icon-check"
                          ></i>
                        </div>
                      </el-col>
                      <el-col :span="3"
                        ><el-button
                          @click="
                            editAttachLabels(
                              labelsList.id,
                              labelsList.name,
                              labelsList.color
                            )
                          "
                          size="mini"
                          icon="el-icon-edit"
                          circle
                        ></el-button
                      ></el-col>
                    </el-row>
                  </div>
                  <el-button plain size="mini" @click="addNexCard"
                    >Tạo nhãn mới</el-button
                  >
                </div>
                <div v-else>
                  <div v-if="checkAttachLabels">
                    <div class="add-new-card-input">
                      <p class="add-new-card-title">Tên</p>
                      <el-input
                        placeholder="Nhập tên nhãn"
                        v-model="addNewCardData"
                      ></el-input>
                    </div>
                    <div class="add-new-card-color">
                      <p class="add-new-card-title">Chọn một màu</p>
                      <div class="add-new-card-color-box">
                        <div style="background-color: #61bd4f">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#61bd4f"
                          />
                        </div>
                        <div style="background-color: #f2d600">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#f2d600"
                          />
                        </div>
                        <div style="background-color: #ff9f1a">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#ff9f1a"
                          />
                        </div>
                        <div style="background-color: #eb5a46">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#eb5a46"
                          />
                        </div>
                        <div style="background-color: #c377e0">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#c377e0"
                          />
                        </div>
                        <div style="background-color: #0079bf">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#0079bf"
                          />
                        </div>
                        <div style="background-color: #00c2e0">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#00c2e0"
                          />
                        </div>
                        <div style="background-color: #51e898">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#51e898"
                          />
                        </div>
                        <div style="background-color: #ff78cb">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#ff78cb"
                          />
                        </div>
                        <div style="background-color: #344563">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#344563"
                          />
                        </div>
                        <div style="background-color: #b3bac5">
                          <input
                            v-model="radioColor"
                            type="radio"
                            name="gender"
                            value="#b3bac5"
                          />
                        </div>
                      </div>
                      <div>
                        <el-button @click="addCard" type="success" size="mini"
                          >Tạo mới</el-button
                        >
                        <el-button
                          type="text"
                          @click="cancalAddCard"
                          size="mini"
                          >Thoát</el-button
                        >
                      </div>
                    </div>
                  </div>
                  <div v-else>
                    <div class="add-new-card-input">
                      <p class="add-new-card-title">Tên</p>
                      <el-input
                        placeholder="Nhập tên nhãn"
                        v-model="addNewCardDataEdit"
                      ></el-input>
                    </div>
                    <div class="add-new-card-color">
                      <p class="add-new-card-title">Chọn một màu</p>
                      <div class="add-new-card-color-box">
                        <div style="background-color: #61bd4f">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#61bd4f"
                          />
                        </div>
                        <div style="background-color: #f2d600">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#f2d600"
                          />
                        </div>
                        <div style="background-color: #ff9f1a">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#ff9f1a"
                          />
                        </div>
                        <div style="background-color: #eb5a46">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#eb5a46"
                          />
                        </div>
                        <div style="background-color: #c377e0">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#c377e0"
                          />
                        </div>
                        <div style="background-color: #0079bf">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#0079bf"
                          />
                        </div>
                        <div style="background-color: #00c2e0">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#00c2e0"
                          />
                        </div>
                        <div style="background-color: #51e898">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#51e898"
                          />
                        </div>
                        <div style="background-color: #ff78cb">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#ff78cb"
                          />
                        </div>
                        <div style="background-color: #344563">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#344563"
                          />
                        </div>
                        <div style="background-color: #b3bac5">
                          <input
                            v-model="radioColorEdit"
                            type="radio"
                            name="gender"
                            value="#b3bac5"
                          />
                        </div>
                      </div>
                      <div>
                        <el-button
                          @click="editLabels"
                          type="success"
                          size="mini"
                          >Sửa</el-button
                        >
                        <el-button
                          @click="deleteLabels"
                          type="danger"
                          size="mini"
                          >Xóa</el-button
                        >
                        <el-button
                          type="text"
                          @click="cancalAddCard"
                          size="mini"
                          >Thoát</el-button
                        >
                      </div>
                    </div>
                  </div>
                </div>
                <el-button class="button-detail-add-tag" slot="reference">
                  <i class="el-icon-price-tag"></i>
                  Nhãn
                </el-button>
              </el-popover>
            </div>
            <div class="title-tag-detail-button">
              <el-popover
                class="labels"
                placement="bottom"
                width="350"
                trigger="click"
              >
                <div class="add-new-card-input">
                  <p class="add-new-card-title">Tiêu đề</p>
                  <el-input
                    placeholder="Nhập tiêu đề công viêc"
                    v-model="addNewCheckListData"
                    @keyup.enter.native="addCheckList"
                  ></el-input>
                </div>
                <div class="add-new-card-color">
                  <div>
                    <el-button @click="addCheckList" type="success" size="mini"
                      >Thêm</el-button
                    >
                  </div>
                </div>
                <el-button class="button-detail-add-tag" slot="reference">
                  <i class="el-icon-document-checked"></i>
                  Việc cần làm
                </el-button>
              </el-popover>
            </div>
            <div class="title-tag-detail-button">
              <el-popover
                class="labels"
                placement="bottom"
                width="350"
                trigger="click"
              >
                <el-date-picker
                  style="width: 100%"
                  v-model="dateValue"
                  type="datetime"
                  placeholder="Chọn ngày hết hạn"
                >
                </el-date-picker>
                <div class="date-list">
                  <el-button @click="addDateList" type="success" size="mini"
                    >Lưu</el-button
                  >
                  <el-button type="danger" size="mini">Gỡ bỏ</el-button>
                </div>
                <el-button class="button-detail-add-tag" slot="reference">
                  <i class="el-icon-time"></i>
                  Ngày hết hạn
                </el-button>
              </el-popover>
            </div>
            <div class="title-tag-detail-button">
              <el-popover
                class="labels"
                placement="bottom"
                width="350"
                trigger="click"
              >
                <div class="file-input">
                  <input @change="changeFile" type="file" id="file" class="file">
                  <span v-if="fileData" @click="fileData = ''">
                    Xóa tệp
                  </span>
                  <label v-else for="file">
                    Chọn tệp
                  </label>
                </div>
                <el-button type="success" @click="addFille">Thêm tệp</el-button>
                <el-button class="button-detail-add-tag" slot="reference">
                  <i class="el-icon-files"></i>
                  Đính kèm
                </el-button>
              </el-popover>
            </div>
          </div>
        </el-col>
      </el-row>
      <span slot="footer" class="dialog-footer">
        <el-tooltip
          class="item"
          effect="dark"
          content="Xóa thẻ"
          placement="left"
        >
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            @click="deleteCard(card.id)"
          ></el-button>
        </el-tooltip>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import Tag from "../lists/Tag";
import axios from "axios";
import moment from "moment";
import CheckListsComponent from "./CheckListsComponent"

export default {
  name: "Home",
  props: ["card", "getdirectories"],
  components: {
    Tag,
    CheckListsComponent
  },
  data() {
    return {
      cardData: [],
      labels: [],
      labelsLists: [],
      checkLists: [],
      fileLists: [],
      dialogFormVisible: false,
      titleClickCheck: true,
      dataTitle: this.card.title,
      titleClickCard: true,
      dataTitleCard: this.card.description,
      colorCard: "pink",
      checkAddCard: true,
      addNewCardData: "",
      radioColor: "#61bd4f",
      addNewCheckListData: "Việc cần làm",
      dateValue: "",
      checkAttachLabels: true,
      idLabel: 0,
      addNewCardDataEdit: "",
      radioColorEdit: "",
      fileData: '',
      checkNameFile: 0,
      dataNameFile: '',
      sheachCards: ''
    };
  },
  watch: {
    sheachCards: function() {
      this.getLabels();
    }
  },
  methods: {
    clickTitleNode() {
      this.titleClickCheck = false;
    },
    outTitleNode() {
      let dataDescription = "";
      if (this.dataTitleCard) {
        dataDescription = this.dataTitleCard;
      }
      axios({
        method: "put",
        url: "http://vuecourse.zent.edu.vn/api/cards/" + this.cardData.id,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          title: this.dataTitle,
          description: dataDescription,
        },
      })
        .then(() => {
          this.getCards();
          this.getdirectories();
          this.titleClickCheck = true;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    clickTitleCard() {
      this.titleClickCard = false;
    },
    outTitleCard() {
      axios({
        method: "put",
        url: "http://vuecourse.zent.edu.vn/api/cards/" + this.cardData.id,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          title: this.dataTitle,
          description: this.dataTitleCard,
        },
      })
        .then(() => {
          this.getCards();
          this.getdirectories();
          this.titleClickCard = true;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    deleteFile(id) {
      this.$confirm("Bạn có chắc chắn muốn xóa?", "Thông báo", {
        confirmButtonText: "OK",
        cancelButtonText: "Hủy",
        type: "warning",
        center: true,
      })
        .then(() => {
          axios({
            method: "delete",
            url: "http://vuecourse.zent.edu.vn/api/files/" + id,
            headers: {
              Authorization: `Bearer ${localStorage.getItem("access_token")}`,
            },
          })
            .then(() => {
              this.getCards();
              this.getdirectories();
            })
            .catch((error) => {
              console.log(error);
            });
          this.$message({
            type: "success",
            message: "Xóa thành công",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Hủy xóa",
          });
        });
    },
    deleteCard(id) {
      this.$confirm("Bạn có chắc chắn muốn xóa?", "Thông báo", {
        confirmButtonText: "OK",
        cancelButtonText: "Hủy",
        type: "warning",
        center: true,
      })
        .then(() => {
          axios({
            method: "delete",
            url: "http://vuecourse.zent.edu.vn/api/cards/" + id,
            headers: {
              Authorization: `Bearer ${localStorage.getItem("access_token")}`,
            },
          })
            .then(() => {
              this.getdirectories();
              this.dialogFormVisible = false;
            })
            .catch((error) => {
              console.log(error);
            });
          this.$message({
            type: "success",
            message: "Xóa thành công",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Hủy xóa",
          });
        });
    },
    addNexCard() {
      this.checkAddCard = false;
    },
    cancalAddCard() {
      this.checkAddCard = true;
      this.checkAttachLabels = true;
    },
    addCard() {
      axios({
        method: "post",
        url:
          "http://vuecourse.zent.edu.vn/api/cards/" + this.card.id + "/label",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          name: this.addNewCardData,
          color: this.radioColor,
        },
      })
        .then(() => {
          this.addNewCardData = "";
          this.radioColor = "#61bd4f";
          this.getCards();
          this.getdirectories();
          this.getLabels();
          this.checkAddCard = true;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    editLabels() {
      axios({
        method: "put",
        url: "http://vuecourse.zent.edu.vn/api/labels/" + this.idLabel,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          name: this.addNewCardDataEdit,
          color: this.radioColorEdit,
        },
      })
        .then(() => {
          this.addNewCardDataEdit = "";
          this.radioColorEdit = "";
          this.getCards();
          this.getdirectories();
          this.getLabels();
          this.checkAddCard = true;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    deleteLabels() {
      axios({
        method: "delete",
        url: "http://vuecourse.zent.edu.vn/api/labels/" + this.idLabel,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
      })
        .then(() => {
          this.getCards();
          this.getdirectories();
          this.getLabels();
          this.checkAddCard = true;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    formatDate(dateString) {
      return moment(dateString).format("YYYY-MM-DD hh:mm:ss");
    },
    formatDate2(dateString) {
      return moment(dateString).format("DD - MM");
    },
    addDateList() {
      axios({
        method: "put",
        url:
          "http://vuecourse.zent.edu.vn/api/cards/" +
          this.card.id +
          "/change-status-deadline",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          deadline: this.formatDate(this.dateValue),
        },
      })
        .then(() => {
          this.statusDateDate();
          this.getCards();
          this.getdirectories();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    statusDateDate() {
      axios({
        method: "put",
        url:
          "http://vuecourse.zent.edu.vn/api/cards/" +
          this.card.id +
          "/change-status",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          status: 0,
        },
      })
        .then(() => {
          this.getCards();
          this.getdirectories();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    checkBoxDateDate() {
      if (this.cardData.status == 1) {
        this.statusDateDate();
      } else {
        axios({
          method: "put",
          url:
            "http://vuecourse.zent.edu.vn/api/cards/" +
            this.card.id +
            "/change-status",
          headers: {
            Authorization: `Bearer ${localStorage.getItem("access_token")}`,
          },
          data: {
            status: 1,
          },
        })
          .then(() => {
            this.getCards();
            this.getdirectories();
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },
    addCheckList() {
      axios({
        method: "post",
        url: "http://vuecourse.zent.edu.vn/api/check-lists",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          title: this.addNewCheckListData,
          card_id: this.card.id,
        },
      })
        .then(() => {
          this.getCards();
          this.getdirectories();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    attachLabels(id) {
      let aa;
      for (let index = 0; index < this.labels.length; index++) {
        if (this.labels[index].id == id) {
          aa = this.labels[index].id;
        }
      }
      if (!aa) {
        axios({
          method: "post",
          url:
            "http://vuecourse.zent.edu.vn/api/cards/" +
            this.card.id +
            "/attach-labels",
          headers: {
            Authorization: `Bearer ${localStorage.getItem("access_token")}`,
          },
          data: {
            label_id: id,
          },
        })
          .then(() => {
            this.getCards();
            this.getdirectories();
            this.getLabels();
          })
          .catch((error) => {
            console.log(error);
          });
      } else {
        axios({
          method: "delete",
          url:
            "http://vuecourse.zent.edu.vn/api/cards/" +
            this.card.id +
            "/detach-labels",
          headers: {
            Authorization: `Bearer ${localStorage.getItem("access_token")}`,
          },
          data: {
            label_id: id,
          },
        })
          .then(() => {
            this.getCards();
            this.getdirectories();
            this.getLabels();
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },
    editAttachLabels(id, name, color) {
      this.idLabel = id;
      this.addNewCardDataEdit = name;
      this.radioColorEdit = color;
      this.checkAddCard = false;
      this.checkAttachLabels = false;
    },
    changeFile(e) {
      if (e.target.files.length) {
        this.fileData = e.target.files[0];
      }
    },
    addFille() {
      if (this.fileData) {
        const formData = new FormData();
          formData.append("file", this.fileData);

          axios({
            method: "post",
            url: "http://vuecourse.zent.edu.vn/api/cards/" +this.card.id +"/upload-file",
            headers: {
              Authorization: `Bearer ${localStorage.getItem("access_token")}`,
            },
            data: formData,
          }).then(() => {
            this.fileData = '';
            this.getCards();
            this.getdirectories();
          }).catch((error) => {
            console.log(error);
          });
      } else {
        this.$notify.error({
          title: 'Lỗi tệp',
          message: 'Tệp không được để trống .Xin kiểm tra lại'
        });
      }
    },
    editFile(id) {
      axios({
        method: "put",
        url: "http://vuecourse.zent.edu.vn/api/files/"+id,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        data: {
          name: this.dataNameFile
        }
      })
        .then(() => {
          this.checkNameFile = 0;
          this.dataNameFile = '';
          this.getCards();
          this.getdirectories();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getLabels() {
      axios({
        method: "get",
        url: "http://vuecourse.zent.edu.vn/api/labels?q="+this.sheachCards,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
      })
        .then((response) => {
          // console.log(response.data.data)
          this.labelsLists = response.data.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getCards() {
      axios({
        method: "get",
        url: "http://vuecourse.zent.edu.vn/api/cards/" + this.card.id,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
      })
        .then((response) => {
          // console.log(response.data.data.files)
          this.cardData = response.data.data;
          this.labels = response.data.data.labels;
          this.checkLists = response.data.data.check_lists;
          this.fileLists = response.data.data.files;
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  mounted() {
    this.getdirectories();
    this.getCards();
    this.getLabels();
  },
};
</script>

<style scoped lang="scss">
.content-tag {
  display: flex;
  flex-wrap: wrap;
  background-color: #fff;
  border-radius: 10px 10px 0 0;
  margin-left: 10px;
  margin-right: 10px;
  padding: 10px;
}
.content-text {
  background-color: #fff;
  margin-left: 10px;
  margin-right: 10px;
  padding-left: 10px;
  padding-right: 10px;
  font-size: 14px;
}
.content-end {
  display: flex;
  flex-wrap: wrap;
  background-color: #fff;
  border-radius: 0 0 10px 10px;
  margin-bottom: 10px;
  margin-left: 10px;
  margin-right: 10px;
  padding: 10px;
  .content-end-detail {
    border-radius: 5px;
    padding: 5px;
    margin-right: 4px;
    margin-bottom: 4px;
    font-size: 14px;
  }
}
</style>

// style dialog
<style scoped>
.left-title {
  font-size: 24px;
}

.right-title {
  margin-left: 13px;
}

.right-title > p {
  font-size: 24px;
  padding-bottom: 5px;
}

.right-title > span {
  font-size: 14px;
  margin-top: 5px;
}

.dialog-container {
  margin-top: 20px;
}

.tag-detail-tag {
  display: flex;
  flex-wrap: wrap;
}

.tag-detail-tag div {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  color: white;
  padding: 10px;
  font-size: 14px;
  border-radius: 5px;
  margin-bottom: 5px;
  margin-right: 5px;
  cursor: pointer;
  transition: 0.5s;
}

.title-tag-detail {
  margin-top: 5px;
  margin-bottom: 5px;
  color: gray;
  font-weight: bold;
}

.button-detail-add-tag {
  width: 100%;
  text-align: left;
}

.title-tag-detail-button {
  margin-bottom: 5px;
}

.magin-mission {
  margin-bottom: 10px;
}

.tag-content {
  padding: 8px;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
  color: white;
  margin-bottom: 10px;
}

.tag-content span {
  text-overflow: ellipsis;
  overflow: hidden;
  margin-right: 10px;
  white-space: nowrap;
}

.tag-content:hover {
  opacity: 0.5;
  cursor: pointer;
}

.add-new-card-input {
  width: 100%;
  margin-bottom: 10px;
}

.add-new-card-title {
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 5px;
}

.add-new-card-color-box {
  display: flex;
  flex-wrap: wrap;
  width: 100%;
  margin-bottom: 10px;
}

.add-new-card-color-box div {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: 35px;
  color: white;
  border-radius: 5px;
  margin-right: 10px;
  margin-bottom: 10px;
}

.add-new-card-color-box div:hover {
  opacity: 0.5;
  cursor: pointer;
}

.date-list {
  margin-top: 10px;
}

.checkbox-date {
  display: flex;
  align-items: center;
}

.checkbox-date div {
  background-color: #eaecf0;
  border-radius: 5px;
  padding: 10px;
  margin-left: 5px;
}

.checkbox-date div span {
  color: white;
  margin: 5px;
  border-radius: 5px;
  padding: 5px;
  font-size: 10px;
}

/* file */
.file-input {
  text-align: center;
  margin-bottom: 15px;
  margin-top: 5px;
}
.file {
  opacity: 0;
  width: 0.1px;
  height: 0.1px;
  position: absolute;
}

.file-input span {
  display: inline-block;
  position: relative;
  width: 200px;
  height: 50px;
  border-radius: 10px;
  background-color: #f5f6f8;
  box-shadow: 0 0 10px 5px #f5f6f8;
  line-height: 50px;
  color: rgb(245, 73, 73);
  font-weight: bold;
  cursor: pointer;
}

.file-input label {
  display: inline-block;
  position: relative;
  width: 200px;
  height: 50px;
  border-radius: 10px;
  background-color: #f5f6f8;
  box-shadow: 0 0 10px 5px #f5f6f8;
  line-height: 50px;
  color: grey;
  font-weight: bold;
  cursor: pointer;
}

.button-file-edit {
  display: flex;
  justify-content: space-between;
}

.file-name {
  font-weight: bold;
  font-size: 12px;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
</style>