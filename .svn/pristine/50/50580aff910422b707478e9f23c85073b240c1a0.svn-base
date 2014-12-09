#include "SingleWordHandAssocController.h"

#include <iostream>
using namespace std;

// 单例指针初始化为空
SingleWordHandAssocController* SingleWordHandAssocController::m_instance = NULL;
bool SingleWordHandAssocController::m_bIsAttached = false;

// 构造
SingleWordHandAssocController::SingleWordHandAssocController():
m_nAssocCount(0),
m_ptAssocBigram(NULL),
m_nPseudoTime(0)
{
	m_ptAssocBigram = new t_SingleHandAssocBigram[ START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT ];
	memset(m_ptAssocBigram, 0, START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT * sizeof(t_SingleHandAssocBigram) );
}

// 析构
SingleWordHandAssocController::~SingleWordHandAssocController()
{
	delete[] m_ptAssocBigram;
}

// 单例
SingleWordHandAssocController* SingleWordHandAssocController::GetInstance()
{
	if (m_instance == NULL)
	{
		m_instance = new SingleWordHandAssocController();
		if(m_instance == NULL)
		{
			//LOG_ERROR("SingleWordHandAssocController::GetInstance Failed");
			
			m_bIsAttached = false;
			return NULL;
		}		
	}

	m_instance->Attach(FILENAME_SINGLE_WORD_ASSOC_BIGRAM);
	
	return m_instance;
}

// 学词
bool SingleWordHandAssocController::LearnAssocBigram(const s_wchar p_cLeft, const s_wchar p_cRight)
{	
	// 空的时候特殊处理
	if ( m_nAssocCount == 0 )	
	{
		// 如果二元数组为空，放到下标为0的第一个位置上
		m_nAssocCount = 1;
		
		SetLeftGram(0, p_cLeft);
		SetRightGram(0, p_cRight);
		ResetFreq(0);
		IncreaseFreq(0);
		StampPseudotime(0);
		return true;
	}
	
	int nPos = FindtPositionForUpdateOrInsert(p_cLeft, p_cRight);
	
	if ( 	GetLeftGram(nPos) == p_cLeft 
		&& 	GetRightGram(nPos) == p_cRight)	
	{
		// 左右元都相同，说明联想二元已学过，需要更新输入次数
		IncreaseFreq(nPos);
	}
	else
	{		
		// 先检查是否达到容量上限，否则考虑扩容
		if ( m_nAssocCount >= MAX_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT )
		{
			// 淘汰
			Eliminate();
			
			// 淘汰后需要重新确定插入位置
			nPos = FindtPositionForUpdateOrInsert(p_cLeft, p_cRight);
		}		
		else if ( m_nAssocCount >= START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT &&
			(m_nAssocCount - START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT) % INCR_STEP_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT == 0 )
		{
			// 扩容
			if(!ExpandVolume())
			{
				// 扩容失败，返回错误
				return false;
			}
		}	

		// 左右元不完全相同，说明是联想二元的插入位置下标，需要移位插入
		memmove(&m_ptAssocBigram[ nPos + 1 ],
				&m_ptAssocBigram[ nPos ],
				(m_nAssocCount - nPos) * sizeof(m_ptAssocBigram[0]) );
				
		SetLeftGram(nPos, p_cLeft);		// 设置左元
		SetRightGram(nPos, p_cRight);	// 设置右元
		ResetFreq(nPos);				// 初始化输入次数为1
		IncreaseFreq(nPos);
		
		m_nAssocCount++;
	}
	StampPseudotime(nPos);				// 盖上伪时间

	return true;
}

// 删词
bool SingleWordHandAssocController::DeleteAssocBigram(const s_wchar p_cLeft, const s_wchar p_cRight)
{
	int nPos = FindtPositionForUpdateOrInsert(p_cLeft, p_cRight);
	
	if ( 	GetLeftGram(nPos) == p_cLeft 
		&& 	GetRightGram(nPos) == p_cRight)	
	{
		// 左右元都相同，找到了要删除的二元
		memmove( m_ptAssocBigram + nPos, m_ptAssocBigram + nPos + 1, ( m_nAssocCount - nPos - 1 ) * sizeof(m_ptAssocBigram[0]) );
		m_nAssocCount--;
	}
	
	return true;
}

// 两对 Getter Settter， 规范对二元项的字段访问
s_wchar SingleWordHandAssocController::GetLeftGram(int p_nIndex)
{
	return m_ptAssocBigram[ p_nIndex ].m_cLeft;
}

s_wchar SingleWordHandAssocController::GetRightGram(int p_nIndex)
{
	return m_ptAssocBigram[ p_nIndex ].m_cRight;
}

void	SingleWordHandAssocController::SetLeftGram(int p_nIndex, s_wchar p_cGram)
{
	m_ptAssocBigram[ p_nIndex ].m_cLeft		= p_cGram;
}

void	SingleWordHandAssocController::SetRightGram(int p_nIndex, s_wchar p_cGram)
{
	m_ptAssocBigram[ p_nIndex ].m_cRight 	= p_cGram;
}

void	SingleWordHandAssocController::IncreaseFreq(int p_nIndex)
{
	++m_ptAssocBigram[ p_nIndex ].m_nFreq;
}

void	SingleWordHandAssocController::ResetFreq(int p_nIndex)
{
	m_ptAssocBigram[ p_nIndex ].m_nFreq = 0;
}


void	SingleWordHandAssocController::StampPseudotime(int p_nIndex)
{
	m_ptAssocBigram[ p_nIndex ].m_nPseudoTime = m_nPseudoTime++;	// 盖上伪时间之后，将控制器伪时间加一
}

// 查词
bool SingleWordHandAssocController::GetSingleWordAssocGrams(const s_wchar p_cLeft, t_SingleHandAssocBigram* p_ptRightArray, int& nResultNum)
{
	// 设置结果集大小为0
	nResultNum = 0;	

	int nPosHitLeft = HitBigramByLeftgramBisect(p_cLeft);
	
	if(nPosHitLeft == -1)
	{
		return false;
	}
	
	int nLeftBorder;
	int nRightBorder;

	// 找左边界和右边界
	nLeftBorder = nPosHitLeft;
	while(nLeftBorder > 0)
	{
		// 如果有相同的左元，继续寻找左边界
		if ( GetLeftGram(nLeftBorder - 1) == GetLeftGram(nLeftBorder) )
		{
			--nLeftBorder;
		}
		else
		{
			break;
		}
	}
	
	// 找左边界和右边界
	nRightBorder = nPosHitLeft;
	while(nRightBorder < m_nAssocCount-1)
	{
		// 如果有相同的左元，继续寻找左边界
		if ( GetLeftGram(nRightBorder + 1) == GetLeftGram(nRightBorder) )
		{
			++nRightBorder;
		}
		else
		{
			break;
		}
	}

	// 输出结果，即右元
	nResultNum = nRightBorder - nLeftBorder + 1;
	int p;
	for( p = nLeftBorder; p <= nRightBorder; p++)
	{		
		p_ptRightArray[ p - nLeftBorder ] = m_ptAssocBigram[p];
	}
	
	// 根据使用次数排序联想右元
	qsort(p_ptRightArray, nResultNum, sizeof(m_ptAssocBigram[0]), FreqCompare4HandAssoc);
	
	return true;
}

// 读取词库
bool SingleWordHandAssocController::Attach(const char* p_szPath)
{
	int fp = open(p_szPath, O_RDONLY);
	if (fp > 0)
	{
		read(fp, &m_nAssocCount, sizeof(m_nAssocCount));
		
		if ( m_nAssocCount > MAX_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT )
		{
			// LOG_ERROR("SingleWordHandAssocController::Attach VALUE TOO LARGE m_nAssocCount = ", m_nAssocCount);
			return false;
		}
		
		if ( m_nAssocCount >= START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT )
		{
			t_SingleHandAssocBigram* pOldBase = m_ptAssocBigram;
			int nCapacity = START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT + 
				(( m_nAssocCount - START_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT ) / INCR_STEP_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT + 1)
				* INCR_STEP_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT;
			m_ptAssocBigram = new t_SingleHandAssocBigram[nCapacity];
			if (!m_ptAssocBigram)
			{
				m_ptAssocBigram = pOldBase;
				return false;
			}
			
			memset(m_ptAssocBigram, 0, m_nAssocCount * sizeof(m_ptAssocBigram[0]) );
			delete[] pOldBase;
		}
		
		read(fp, &m_nPseudoTime, sizeof(m_nPseudoTime));
		
		read(fp, m_ptAssocBigram, sizeof(m_ptAssocBigram[0]) * m_nAssocCount);
		close(fp);
		m_bIsAttached = true;
		return true;
	}
	else
	{
		return false;
	}
}

// 回写词库
bool SingleWordHandAssocController::Save(const char* p_szPath)
{
	int fd = open(p_szPath, O_RDWR | O_CREAT, S_IRWXU | S_IRWXG | S_IRWXO );

	write(fd, &m_nAssocCount, sizeof(m_nAssocCount));
	write(fd, &m_nPseudoTime, sizeof(m_nPseudoTime));	
	write(fd, m_ptAssocBigram, sizeof(m_ptAssocBigram[0]) * m_nAssocCount);

	close(fd);

	return true;
}


// 二元比较函数，用于查找二元
unsigned int SingleWordHandAssocController::ComparePosition(const s_wchar p_cLeft1, const s_wchar p_cRight1, const s_wchar p_cLeft2, const s_wchar p_cRight2)
{
	if ( p_cLeft1 != p_cLeft2 )
	{
		return p_cLeft1 - p_cLeft2;
	}
	else
	{
		return p_cRight1 - p_cRight2;
	}	
}

// 在二元数组中为传入二元找插入位置或者已有项的位置
int SingleWordHandAssocController::FindtPositionForUpdateOrInsert(const s_wchar p_cLeft, const s_wchar p_cRight)
{
	int min = 0;
	int max = m_nAssocCount - 1;
	if ( max < 0 )
	{
		return -1;
	}

	int m;
	int nCmpCode;
	
	while(min <= max)
	{
		m = ( min + max ) / 2;
		nCmpCode = ComparePosition(p_cLeft, p_cRight, GetLeftGram(m),GetRightGram(m));
		if(nCmpCode < 0)
		{
			max = m - 1;
		}
		else if(nCmpCode > 0)
		{
			min = m + 1;
		}
		else
		{
			break;
		}
	}
	
	if (nCmpCode > 0)
	{
		return min;
	}
	else if(nCmpCode < 0)
	{
		return m;
	}
	else
	{
		return m;
	}
}


// 按左元做二分命中
int SingleWordHandAssocController::HitBigramByLeftgramBisect(const s_wchar p_cLeft)
{
	int min = 0;
	int max = m_nAssocCount - 1;
	int m;

	// 通过二分命中一个左元
	int nPosHitLeft = -1;
	s_wchar mLeft;
	
	while(min <= max)
	{
		m = (min + max) / 2;
		mLeft	= GetLeftGram(m);
		
		if ( p_cLeft < mLeft )
		{
			max = m - 1;			
		}
		else if ( p_cLeft > mLeft )
		{
			min = m + 1;
		}
		else
		{
			nPosHitLeft = m;
			break;
		}
	}
	
	// 存在查找项返回命中的下标，否则返回无效下标-1
	return nPosHitLeft;
}

bool SingleWordHandAssocController::ExpandVolume()
{
	// 按步进值增大词条数
	int expandedSize = m_nAssocCount + INCR_STEP_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT;

	// 保护
	if (expandedSize < 0)
		return false;
	
	// 申请新的更大的内存
	t_SingleHandAssocBigram* pOldBase = m_ptAssocBigram;
	m_ptAssocBigram = new t_SingleHandAssocBigram[expandedSize];
	
	// 申请大内存失败
	if ( !m_ptAssocBigram )
	{
		// 申请内存失败恢复指针
		m_ptAssocBigram = pOldBase;
		// LOG_ERROR("SingleWordHandAssocController::Expand expandedSize = ", expandedSize);
		return false;
	}
	
	// 清空新内存的数据区
	memset(m_ptAssocBigram, 0, expandedSize * sizeof(m_ptAssocBigram[0]));
	
	// 复制当前的m_nAssocCount个二元数据到新的内存，释放原来的内存
	memcpy(m_ptAssocBigram, pOldBase, m_nAssocCount * sizeof(m_ptAssocBigram[0]) );	
	delete[] pOldBase;	
	
	return true;
}

void SingleWordHandAssocController::Eliminate()
{
	if ( m_nAssocCount >= MAX_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT )
	{
		const int nPostEliminationTotal = 0.75 * MAX_SINGLE_WORD_HAND_ASSOC_BIGRAM_COUNT;
		
		qsort(m_ptAssocBigram, m_nAssocCount, sizeof(m_ptAssocBigram[0]), EliminateCompare4HandAssoc);
		
		m_nAssocCount = nPostEliminationTotal;
		
		qsort(m_ptAssocBigram, m_nAssocCount, sizeof(m_ptAssocBigram[0]), UnicodeCompare4HandAssoc);
		
		// LOG_DEBUG("SingleWordHandAssocController::Eliminate Elimination Completed. m_nAssocCount = ", m_nAssocCount);
	}
}

int main()
{
	SingleWordHandAssocController* inst = SingleWordHandAssocController::GetInstance();
	// for(int i = 19; i >= 10; i--)
		// for(int j = 19; j >= 10; j--)
		
	// 单条删词测试
	// inst->DeleteAssocBigram(5999,59999);
	// goto PRINT_RESULT;
	
	inst->LearnAssocBigram(0x4FAF,0x6668);
	inst->LearnAssocBigram(0x6668,0x66E6);
	
	if(false)
	for(int i = 0; i < 6000; i++)
		for(int j = i*10; j < i*10 + 10; j++)
		{
			cout << i << "," << j << endl;
			
			for(int k = 0; k < (j % 5) + 1; k++)
				inst->LearnAssocBigram(i,j);
		}

	// 删词测试
	DELETE_TEST:
	if(false)
	for(int i = 5999; i >=0 ; i--)
		for(int j = i*10 + 9; j >= i*10 ; j--)
		{
			inst->DeleteAssocBigram(i,j);
		}
		

	t_SingleHandAssocBigram array[1000];
	int num;

	PRINT_RESULT:
	if(false)
	for(int i = 0; i < 6000; i++)
	{
		inst->GetSingleWordAssocGrams(i,array,num);
		if ( 0 == num )
			continue;
			
		cout << "[" << i << "]"  << "[" << num << "]" << "\t" ;
		for(int j = 0; j < num; j++)
			cout << array[j].m_cRight /*<< "(" << array[j].m_nFreq << "," << array[j].m_nPseudoTime << ")"*/ << "\t";
		cout << endl;
	}
	
	inst->Save(FILENAME_SINGLE_WORD_ASSOC_BIGRAM);
}
