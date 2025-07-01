# Korean Translation of 500 Lines or Less

## 📋 Project Overview

This repository contains the Korean translation of "500 Lines or Less" ebook. The project maintains the original English content while adding Korean translations in a separate directory structure to enable upstream merging.

**Main Project Tracking**: [GitHub Issue #1](https://github.com/warmblood-kr/500lines-ko/issues/1)

## 🚀 Quick Start

### Building Korean Version
```bash
# Install dependencies
pip install -r requirements.txt

# Build Korean ebook
LANGUAGE=ko python3 build.py --epub

# Build single chapter (for testing)
LANGUAGE=ko python3 build.py web-server --epub

# Build English (original, unchanged)
python3 build.py --epub
```

### Output Files
- Korean: `output/500L.epub`
- Preserves original English build functionality

## 📁 Directory Structure

```
500lines-ko/
├── translations/ko/              # 🇰🇷 Korean translations
│   ├── web-server/
│   │   ├── web-server.markdown   # ← Korean content here
│   │   └── web-server-images/    # Images (copied from original)
│   ├── interpreter/
│   ├── template-engine/
│   └── ... (22 chapters total)
├── web-server/                   # 🇺🇸 Original English (untouched)
├── interpreter/                  # 🇺🇸 Original English (untouched)
├── build.py                      # Modified for language support
└── CLAUDE.md                     # This file
```

## 🎯 Translation Progress

### Current Status: **Infrastructure Complete (5%)**

**✅ Completed:**
- Multi-language build system
- All 22 chapters copied to Korean directory
- Cross-platform compatibility
- Python 2/3 build system fixes

**🔄 Translation Priority:**
1. **web-server** - HTTP fundamentals
2. **interpreter** - Programming language basics  
3. **template-engine** - Text processing
4. **crawler** - Async programming
5. **data-store** - Database concepts

**📈 Progress Tracking**: See [Issue #1](https://github.com/warmblood-kr/500lines-ko/issues/1) for detailed checklist

## 🛠️ Technical Implementation

### Language Support System
The build system detects language via `LANGUAGE` environment variable:
- `LANGUAGE=en` (default): Uses original English content
- `LANGUAGE=ko`: Uses Korean translations from `translations/ko/`

### Key Changes Made
1. **build.py modifications**: Added language path resolution
2. **Directory structure**: Additive approach (no original files modified)
3. **Image handling**: Copy-based (no symlinks for Windows compatibility)
4. **Python compatibility**: Fixed Python 2→3 print statements

### Upstream Merge Strategy
- ✅ **Zero conflicts**: Original content untouched
- ✅ **Additive only**: New files in `translations/` directory
- ✅ **Backwards compatible**: English builds unchanged
- ✅ **Minimal changes**: <20 lines of build system modifications

## 🤝 Collaboration Guidelines

### For Translation Work
1. **Focus on content**: Only translate `.markdown` files
2. **Preserve structure**: Keep all markdown formatting
3. **Maintain code**: Leave code examples in original language
4. **Test builds**: Run `LANGUAGE=ko python3 build.py [chapter] --epub` after changes
5. **Commit frequently**: Small, focused commits with clear messages

### Branch Strategy
- `master`: Stable with working build system
- Feature branches: `translate/[chapter-name]` for individual chapters
- Testing branches: `test/[feature]` for build system experiments

### Commit Message Format
```
[chapter]: Brief description of changes

- Detailed change 1
- Detailed change 2

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## 📚 Translation Guidelines

### Korean Programming Terminology
- **Algorithm** → 알고리즘
- **Class** → 클래스  
- **Database** → 데이터베이스
- **Function** → 함수
- **Object** → 객체
- **Server** → 서버
- **Variable** → 변수

### Quality Standards
1. **Technical accuracy** over literal translation
2. **Consistent terminology** across all chapters
3. **Preserve author intent** and voice
4. **Natural Korean flow** while maintaining technical precision
5. **Code preservation** - Never translate code examples

### Files to Translate
**Primary targets**: `{chapter}/{chapter}.markdown` files
**Leave unchanged**: All code files, images, README files

## 🧰 Development Environment

### Required Tools
- **Python 3.x** (tested with 3.11+)
- **pandoc** (for markdown processing)
- **LaTeX** (for PDF generation, optional)
- **Git & GitHub CLI** (for collaboration)

### Setup Commands
```bash
# Clone repository
git clone https://github.com/warmblood-kr/500lines-ko.git
cd 500lines-ko

# Install dependencies  
pip install -r requirements.txt

# Test Korean build
LANGUAGE=ko python3 build.py web-server --epub
```

### Troubleshooting
- **Python command issues**: Use `python3` explicitly
- **Permission errors**: Ensure write access to `output/` directory
- **Pandoc errors**: Check pandoc version (≥2.0 recommended)
- **Build failures**: Check [Issue #1](https://github.com/warmblood-kr/500lines-ko/issues/1) for known issues

## 📊 Project Metrics

### Repository Statistics
- **Total chapters**: 22
- **Lines of content**: ~146,000+ (including code)
- **Translation completion**: 5% (infrastructure only)
- **Estimated effort**: 3-6 months for full translation

### Build System Coverage
- ✅ **EPUB generation**: Working
- ⚠️ **PDF generation**: Needs testing with Korean fonts
- ⚠️ **HTML generation**: Needs testing
- ❌ **MOBI generation**: Pending

## 🎯 Next Session Goals

### Immediate Tasks (Next 1-2 Sessions)
1. **Start web-server translation** - Most foundational chapter
2. **Test complete build pipeline** - Ensure Korean fonts work in PDF
3. **Establish translation rhythm** - Small, consistent progress

### Medium-term Goals (Next Month)
1. **Complete 3-5 high-priority chapters**
2. **Refine build system** - Fix any Korean-specific issues
3. **Documentation improvements** - Based on translation experience

### Long-term Vision (3-6 Months)
1. **Complete all 22 chapters**
2. **Quality review process** - Technical accuracy validation
3. **Community engagement** - Share with Korean developer community

---

**Last Updated**: 2025-07-01  
**Current Session**: Infrastructure Setup Complete  
**Next Priority**: Begin web-server chapter translation

For detailed progress tracking and task management, see: [GitHub Issue #1](https://github.com/warmblood-kr/500lines-ko/issues/1)