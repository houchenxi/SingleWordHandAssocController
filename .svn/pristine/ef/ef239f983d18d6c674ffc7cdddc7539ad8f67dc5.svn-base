#ifndef T_SINGLE_WORD_ASSOC_H
#define T_SINGLE_WORD_ASSOC_H

#define FILENAME_SINGLE_WORD_ASSOC_BIGRAM "sgim_swab.bin"
#define START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT 		5000
#define INCR_STEP_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT	2000
#define MAX_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT 		50000
#include <cstring>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>

typedef wchar_t s_wchar;	//！！！！！！！！！！！！！！！！！！standalone开发使用的typedef，组入时删除！！！！！！！！！！！！！！！！

//#pragma pack(1)
// 单字手写联想二元的结构体，只包含左右元单字的Unicode值和输入次数
struct t_SingleHandAssocBigram
{
	s_wchar m_cLeft;		// 左元unicode
	s_wchar m_cRight;		// 右元unicode	
	short	m_nFreq;		// 联想二元的输入次数
	unsigned int m_nPseudoTime;	// 伪时间
};
//#pragma pack()

// 用于右元按照输入次数排序的比较函数，用于查词时给查出的右元排序
int FreqCompare4HandAssoc(const void* item1, const void* item2)
{	
	t_SingleHandAssocBigram* pItem1 = (t_SingleHandAssocBigram*)item1;
	t_SingleHandAssocBigram* pItem2 = (t_SingleHandAssocBigram*)item2;
	
	int nFreqDiff = pItem2->m_nFreq - pItem1->m_nFreq;
	return (nFreqDiff != 0) ? nFreqDiff : (pItem1->m_cRight - pItem2->m_cRight);
}

// 用于容量达到上限时对词条进行淘汰
int EliminateCompare4HandAssoc(const void* item1, const void* item2)
{
	t_SingleHandAssocBigram* pItem1 = (t_SingleHandAssocBigram*)item1;
	t_SingleHandAssocBigram* pItem2 = (t_SingleHandAssocBigram*)item2;
	
	int nFreqDiff	= pItem2->m_nFreq - pItem1->m_nFreq;
	int nPtimeDiff	= pItem2->m_nPseudoTime - pItem1->m_nPseudoTime;
	return (nFreqDiff != 0) ? nFreqDiff : (
				(nPtimeDiff != 0) ? nPtimeDiff : (pItem1->m_cRight - pItem2->m_cRight)
			);	
}

// 用于容量达到上限，淘汰排序后恢复原有的左元-右元key字典顺序
int UnicodeCompare4HandAssoc(const void* item1, const void* item2)
{
	t_SingleHandAssocBigram* pItem1 = (t_SingleHandAssocBigram*)item1;
	t_SingleHandAssocBigram* pItem2 = (t_SingleHandAssocBigram*)item2;

	int nLeftUnicodeDiff = pItem1->m_cLeft - pItem2->m_cLeft;
	int nRightUnicodeDiff = pItem1->m_cRight - pItem2->m_cRight;
	
	return ( nLeftUnicodeDiff != 0 ) ? nLeftUnicodeDiff : nRightUnicodeDiff;
}

// 单字手写联想功能的控制器类，单例运行
class SingleWordHandAssocController
{
	public:
		// 单例
		static SingleWordHandAssocController* GetInstance();
		// 学词
		bool LearnAssocBigram(const s_wchar p_cLeft, const s_wchar p_cRight);
		// 查词
		bool GetSingleWordAssocGrams(const s_wchar p_cLeft, t_SingleHandAssocBigram* p_ptRightArray, int& nResultNum);
		// 删词
		bool DeleteAssocBigram(const s_wchar p_cLeft, const s_wchar p_cRight);
		
		// 读取词库
		bool Attach(const char* p_szPath);
		// 回写词库
		bool Save(const char* p_szPath);
	
	private:
	
		// 构造函数
		SingleWordHandAssocController();
		~SingleWordHandAssocController();
		
		// Getter Setter 用于按下标对二元数组中每个二元的各个字段进行操作	
		s_wchar GetLeftGram(int p_nIndex);
		s_wchar GetRightGram(int p_nIndex);
		void	SetLeftGram(int p_nIndex, s_wchar p_cGram);
		void	SetRightGram(int p_nIndex, s_wchar p_cGram);
		void	StampPseudotime(int p_nIndex);
		// 输入次数函数
		void	IncreaseFreq(int p_nIndex);
		void	ResetFreq(int p_nIndex);
		
		// 学词找插入位置
		int FindtPositionForUpdateOrInsert(const s_wchar p_cLeft, const s_wchar p_cRight);
		int HitBigramByLeftgramBisect(const s_wchar p_cLeft);
		unsigned int ComparePosition(const s_wchar p_cLeft1, const s_wchar p_cRight1, const s_wchar p_cLeft2, const s_wchar p_cRight2);
		
		// 扩容
		bool ExpandVolume();
		// 淘汰
		void Eliminate();
		
		// 词条数和二元的数组以及伪时间
		int 						m_nAssocCount;
		t_SingleHandAssocBigram*	m_ptAssocBigram;
		unsigned int				m_nPseudoTime;
		
		// 控制器单例的变量
		static bool								m_bIsAttached;
		static SingleWordHandAssocController* 	m_instance;
};
#endif
