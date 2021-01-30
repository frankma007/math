<template>
  <div>
    <p class="f-t">学情报告</p>
    <div>
      <span class="d-s">
        <img @click="toPlateAna" src="@/assets/finemicro/plateana.png" alt="">
        <p>版块环比分析</p>
      </span>
      <span class="d-s">
        <img @click="toClassTraitana" src="@/assets/finemicro/traitana.png" alt="">
        <p>班级特点分析</p>
      </span>
      <span class="d-s">
        <img @click="toUnitana" src="@/assets/finemicro/unitana.png" alt="">
        <p>单元质量分析</p>
      </span>
      <span class="d-s">
        <img @click="toStudyResult" src="@/assets/finemicro/achievement.png" alt="">
        <p>学习成绩分布</p>
      </span>
      <span class="d-s">
        <img @click="toWeakPointerAna" src="@/assets/finemicro/weakpointer.png" alt="">
        <p>薄弱点分析</p>
      </span>
      <span class="d-s">
        <img @click="toHorizontal" src="@/assets/finemicro/horizontal-section.png" alt="">
        <p>水平及版块质量分析</p>
      </span>
    </div>
  </div>
</template>

<script>
export default {
  created () {

  },
  data () {
    return {

    }
  },
  methods: {
    // 版块环比分析
    toPlateAna () {
      // this.$router.push({ path: '/Teacher/TeachingEvaluation/PlateAnalysis' })
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/PlateAnalysisSchool' })
    },
    // 班级特点分析
    toClassTraitana () {
      // this.$router.push({ path: '/Teacher/TeachingEvaluation/StudentTraitana' })
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/ClassTraitanaSchool' })
    },
    // 单元质量分析
    toUnitana () {
      // this.$router.push({ path: '/Teacher/TeachingEvaluation/UnitAnalysis' })
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/UnitAnalysisSchool' })
    },
    // 学习成绩分布
    toStudyResult () {
      // this.$router.push({ path: '/Teacher/TeachingEvaluation/StudyResult' })
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/GradeResultSchool' })
    },
    // 薄弱点分析
    toWeakPointerAna () {
      // this.$router.push({ path: '/Teacher/TeachingEvaluation/WeakPointer' })
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/WeakPointerSchool' })
    },
    // 水平及版块质量分析
    toHorizontal () {
      this.$router.push({ path: '/SchoolManager/DepthAnalysis/HorizontalSection' })
    }
  }
}
</script>

<style lang="less" scoped>
  .d-s {
    display: inline-block;
    text-align: center;
    margin-right: 15px;
    margin-bottom: 30px;
    cursor: pointer;
  }
  .f-t {
    font-size: 22px;
    margin-bottom: 30px;
  }
</style>
