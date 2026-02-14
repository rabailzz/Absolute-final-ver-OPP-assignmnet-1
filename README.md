

//-------------------------------------------- -//QUESTION 6 -----------------------------------

#pragma once
#include <iostream>
#include <fstream>
#include <string>
using namespace std;
float packetnum1(unsigned char* stream, int targetID, int numberOfUnits);
void packetnum2(unsigned char* stream, int size);
void packetnum3(unsigned char* stream, int size);
void processStream(unsigned char* stream)
{
    if (*stream == 255) return;
    int* size = (int*)(stream + 1);
    int numberOfUnits = 0;
    if (*stream == 1) {
        int payload = *size - 5;
        numberOfUnits = (payload - 4) / 12;
        int* targetID = (int*)(stream + 5 + numberOfUnits * 12);
        float result = 0;
        result = packetnum1(stream + 5, 0, numberOfUnits);
        float targetPower = packetnum1(stream + 5, *targetID, numberOfUnits);
        cout << "Necrotic Power: " << (int)targetPower << endl;
    }
    else if (*stream == 2) {
        packetnum2((unsigned char*)(stream + 5), *size - 5);
    }
    else if (*stream == 3) {
        packetnum3(stream + 5, *size - 5);
    }
    processStream(stream + *size);
}
float packetnum1(unsigned char* stream, int targetID, int numberOfUnits)
{
    if (numberOfUnits == 0) return 0;
    unsigned char* unit = stream;
    int* unitID = (int*)(unit + 0);
    float* power = (float*)(unit + 4);
    int* masterID = (int*)(unit + 8);
    float sum = 0;
    if (*unitID == targetID)
    {
        sum += *power;
    }
    if (*masterID == targetID)
    {
        sum += *power;
        sum += packetnum1(unit + 12, *unitID, numberOfUnits - 1);
    }
    sum += packetnum1(unit + 12, targetID, numberOfUnits - 1);
    return sum;
}
int rowsreturn(int size, int rows = 1)
{
    if (rows * rows > size) return 0;
    if (rows * rows == size) return rows;
    return rowsreturn(size, rows + 1);
}
void findStart(unsigned char* flatarray, int rows, int cols, int i, int j, int& startRow, int& startCol)
{
    if (i >= rows) return;
    unsigned char* twodflat = flatarray + i * cols + j;
    if (*twodflat == 'S') {
        startRow = i;
        startCol = j;
        return;
    }
    if (j + 1 < cols)
        findStart(flatarray, rows, cols, i, j + 1, startRow, startCol);
    else
        findStart(flatarray, rows, cols, i + 1, 0, startRow, startCol);
}
bool solveMaze(unsigned char* flatarray, int rows, int cols, int row, int col)
{
    if (row < 0 || row >= rows || col < 0 || col >= cols)
        return false;
    unsigned char* cell = flatarray + row * cols + col;
    if (*cell == '#' || *cell == '*')
        return false;
    if (*cell == 'E')
        return true;
    unsigned char original = *cell;
    if (original != 'S') *cell = '*';
    if (solveMaze(flatarray, rows, cols, row - 1, col)) return true;
    if (solveMaze(flatarray, rows, cols, row + 1, col)) return true;
    if (solveMaze(flatarray, rows, cols, row, col - 1)) return true;
    if (solveMaze(flatarray, rows, cols, row, col + 1)) return true;
    if (original != 'S') *cell = '.';
    return false;
}
void printMaze(unsigned char* flatarray, int rows, int cols, int i = 0, int j = 0)
{
    if (i >= rows) return;
    unsigned char* cell = flatarray + i * cols + j;
    cout << *cell;
    if (j + 1 < cols)
        printMaze(flatarray, rows, cols, i, j + 1);
    else {
        cout << endl;
        printMaze(flatarray, rows, cols, i + 1, 0);
    }
}
void packetnum2(unsigned char* stream, int size)
{
    int rows = rowsreturn(size);
    int cols = rows;
    int startRow = 0;
    int startCol = 0;
    findStart(stream, rows, cols, 0, 0, startRow, startCol);
    bool solved = solveMaze(stream, rows, cols, startRow, startCol);
    if (solved) {
        cout << "Frostbite Labyrinth: SOLVABLE" << endl;
        printMaze(stream, rows, cols);
    }
    else {
        cout << "Frostbite Labyrinth: UNSOLVABLE" << endl;
    }
}
void packetnum3(unsigned char* payload, int size)
{
    int* sampleRate = (int*)(payload);
    int* channelCount = (int*)(payload + 4);
    unsigned char* audioData = payload + 8;
    int numSamples = (size - 8) / 4;
    int stride = *channelCount;
    float* tempData = new float[numSamples];
    for (int i = 0; i < numSamples; i++)
    {
        float sum = 0;
        int count = 0;
        if (i - stride >= 0)
        {
            sum += *((float*)(audioData + (i - stride) * sizeof(float)));
            count++;
        }
        sum += *((float*)(audioData + i * sizeof(float)));
        count++;
        if (i + stride < numSamples)
        {
            sum += *((float*)(audioData + (i + stride) * sizeof(float)));
            count++;
        }
        *(tempData + i) = sum / count;
    }
    for (int i = 0; i < numSamples; i++)
    {
        *((float*)(audioData + i * sizeof(float))) = *(tempData + i);
    }
    delete[] tempData;
    cout << "The Scream: Audio Processed" << endl;
}





//---------------------------------------------------- -//QUESTION 1--------------------------------------------------------------------

int*** CreateViews(int* data, int size, int& num_views) {
    num_views = 0;

    for (int i = 1; i <= size; i++) {
        for (int j = 1; j <= size; j++) {
            if (i * j == size)
                num_views++;
        }
    }


    int*** views = new int** [num_views];

    int count = 0;


    for (int i = 1; i <= size; i++) {
        for (int j = 1; j <= size; j++) {
            if (i * j == size) {

                *(views + count) = new int* [i];

                for (int c = 0; c < i; c++) {
                    *(*(views + count) + c) = data + c * j;
                }

                count++;
            }
        }
    }

    return views;
}



//---------------------------------------------------------------QUESTION 5-------------------------------------------
//(3 failed test cases)

const char* findingMorse(char c) {
    switch (c) {
    case 'a': return ".-";
    case 'b': return "-...";
    case 'c': return "-.-.";
    case 'd': return "-..";
    case 'e': return ".";
    case 'f': return "..-.";
    case 'g': return "--.";
    case 'h': return "....";
    case 'i': return "..";
    case 'j': return ".---";
    case 'k': return "-.-";
    case 'l': return ".-..";
    case 'm': return "--";
    case 'n': return "-.";
    case 'o': return "---";
    case 'p': return ".--.";
    case 'q': return "--.-";
    case 'r': return ".-.";
    case 's': return "...";
    case 't': return "-";
    case 'u': return "..-";
    case 'v': return "...-";
    case 'w': return ".--";
    case 'x': return "-..-";
    case 'y': return "-.--";
    case 'z': return "--..";
    case 'A': return ".-";
    case 'B': return "-...";
    case 'C': return "-.-.";
    case 'D': return "-..";
    case 'E': return ".";
    case 'F': return "..-.";
    case 'G': return "--.";
    case 'H': return "....";
    case 'I': return "..";
    case 'J': return ".---";
    case 'K': return "-.-";
    case 'L': return ".-..";
    case 'M': return "--";
    case 'N': return "-.";
    case 'O': return "---";
    case 'P': return ".--.";
    case 'Q': return "--.-";
    case 'R': return ".-.";
    case 'S': return "...";
    case 'T': return "-";
    case 'U': return "..-";
    case 'V': return "...-";
    case 'W': return ".--";
    case 'X': return "-..-";
    case 'Y': return "-.--";
    case 'Z': return "--..";
    default: return "";
    }
}
bool** refineSignal(const char* inputDirective, int& outWordCount, int*& outWordSizes) {
    if (!inputDirective || *inputDirective == '\0') {
        outWordCount = 0;
        outWordSizes = nullptr;
        return nullptr;
    }

    outWordCount = 0;
    const char* ptr = inputDirective;
    while (*ptr != '\0') {
        if (*ptr != ' ' && (ptr == inputDirective || *(ptr - 1) == ' ')) {
            outWordCount++;
        }
        ptr++;
    }

    outWordSizes = new int[outWordCount];
    bool** clusters = new bool* [outWordCount];

    ptr = inputDirective;
    for (int w = 0; w < outWordCount; w++) {
        while (*ptr == ' ') ptr++;
        const char* wordStart = ptr;

        while (*ptr != '\0' && *ptr != ' ') ptr++;
        const char* wordEnd = ptr;

        int boolSize = 0;
        const char* letterPtr = wordStart;
        while (letterPtr != wordEnd) {
            const char* morse = findingMorse(*letterPtr);
            while (*morse != '\0') {
                if (*morse == '-') boolSize += 3;
                else if (*morse == '.') boolSize += 1;
                morse++;
            }
            letterPtr++;
        }

        int numLetters = 0;
        letterPtr = wordStart;
        while (letterPtr != wordEnd) {
            numLetters++;
            letterPtr++;
        }
        if (numLetters > 1) {
            boolSize += (numLetters - 1) * 2;
        }

        int originalSize = boolSize;
        outWordSizes[w] = boolSize;

        while (boolSize % 16 != 0) {
            boolSize++;
        }

        clusters[w] = new bool[boolSize];
        bool* boolPtr = clusters[w];

        letterPtr = wordStart;
        while (letterPtr != wordEnd) {
            const char* morse = findingMorse(*letterPtr);
            while (*morse != '\0') {
                if (*morse == '-') {
                    *boolPtr++ = true;
                    *boolPtr++ = true;
                    *boolPtr++ = true;
                }
                else if (*morse == '.') {
                    *boolPtr++ = true;
                }
                morse++;
            }

            if (letterPtr + 1 != wordEnd) {
                *boolPtr++ = false;
                *boolPtr++ = false;
            }
            letterPtr++;
        }

        for (int i = originalSize; i < boolSize; i++) {
            *boolPtr++ = false;
        }
    }

    return clusters;
}

long long computeChecksum(bool** cluster, int* sizes, int wordCount, int index)
{
    if (index == wordCount) return 0;
    bool* bits = *(cluster + index);
    unsigned short* ptr = (unsigned short*)bits;

    unsigned short* startptr = ptr;
    unsigned short* endptr = startptr + ((sizes[index] + 15) / 16);
    long long currentVal = 0;

    while (startptr < endptr)
    {
        if (*startptr % 2 == 0)
        {
            currentVal = (currentVal << 1) + (*startptr);
        }

        if (*startptr % 2 != 0)
        {
            currentVal = (currentVal >> 1) ^ (*startptr);
        }

        startptr++;
    }

    if (index % 3 == 0)
    {
        return (currentVal + computeChecksum(cluster, sizes, wordCount, index + 1));
    }
    else
    {
        return (currentVal - computeChecksum(cluster, sizes, wordCount, index + 1));
    }
}
void destroyCluster(bool** cluster, int* sizes, int wordCount)
{
    for (int i = 0;i < wordCount;i++)
    {
        delete[] cluster[i];
    }
    delete[] cluster;
    delete[] sizes;
}



//--------------------------------------------------------------------QUESTION 4 -------------------------------
/*#pragma once
#include"iostream"
#include"fstream"
using namespace std;*/
void getpassword(char* ch, int i = 0)
{
    if (i >= 8) return;
    cout << "Enter character " << i + 1 << ": ";
    cin >> *(ch + i);
    getpassword(ch, i + 1);
}
unsigned char findkey(char* ch)
{
    unsigned char key = 0;
    key = key | ((*(ch + 0) & 1) << 0);
    key = key | ((*(ch + 1) & 1) << 1);
    key = key | ((*(ch + 2) & 1) << 2);
    key = key | ((*(ch + 3) & 1) << 3);
    key = key | ((*(ch + 4) & 1) << 4);
    key = key | ((*(ch + 5) & 1) << 5);
    key = key | ((*(ch + 6) & 1) << 6);
    key = key | ((*(ch + 7) & 1) << 7);
    return key;
}
unsigned char findkeymod(char* ch)
{
    unsigned char keymod = 0;
    keymod = keymod | (((*(ch + 0) & (1 << 1)) >> 1) << 0);
    keymod = keymod | (((*(ch + 1) & (1 << 1)) >> 1) << 1);
    keymod = keymod | (((*(ch + 2) & (1 << 1)) >> 1) << 2);
    keymod = keymod | (((*(ch + 3) & (1 << 1)) >> 1) << 3);
    keymod = keymod | (((*(ch + 4) & (1 << 1)) >> 1) << 4);
    keymod = keymod | (((*(ch + 5) & (1 << 1)) >> 1) << 5);
    keymod = keymod | (((*(ch + 6) & (1 << 1)) >> 1) << 6);
    keymod = keymod | (((*(ch + 7) & (1 << 1)) >> 1) << 7);
    return keymod;
}
int countFileSize(ifstream& file, int count = 0)
{
    char ch;
    if (!file.get(ch)) return count;
    if (ch == '\n') return count;
    return countFileSize(file, count + 1);
}
void storeinbuffer(ifstream& file, char* encBuffer, int index = 0)
{
    char ch;
    if (!file.get(ch)) return;
    if (ch == '\n') return;
    *(encBuffer + index) = ch;
    storeinbuffer(file, encBuffer, index + 1);
}
void decryption(char* encBuffer, char* decBuffer, int currentIndex, unsigned char key, int& decIndex)
{
    int base = currentIndex * 4;
    unsigned char nextchar = (unsigned char)*(encBuffer + (base + 0));
    unsigned char keymodifier = (unsigned char)*(encBuffer + (base + 1));
    char encchar = *(encBuffer + (base + 3));
    *(decBuffer + decIndex) = encchar ^ key;
    decIndex++;
    key = key ^ keymodifier;
    if (nextchar == 255) {
        *(decBuffer + decIndex) = '\0';
        return;
    }
    decryption(encBuffer, decBuffer, nextchar, key, decIndex);
}
void printDecrypted(char* decBuffer, int index = 0)
{
    if (*(decBuffer + index) == '\0') return;
    cout << *(decBuffer + index);
    printDecrypted(decBuffer, index + 1);
}
void processFile(char* buffer, unsigned char key)
{
    ifstream file(buffer);
    if (!file) {
        return;
    }
    int size = countFileSize(file);
    if (size == 0) {
        file.close();
        return;
    }
    file.clear();
    file.seekg(0);
    char* encBuffer = new char[size + 1];
    storeinbuffer(file, encBuffer, 0);
    *(encBuffer + size) = '\0';
    char* decBuffer = new char[size / 4 + 1];
    int middleIndex = size / 8;
    int decIndex = 0;
    decryption(encBuffer, decBuffer, middleIndex, key, decIndex);
    cout << "File: " << buffer << "\nDecrypted: ";
    printDecrypted(decBuffer);
    cout << "\n\n";
    delete[] encBuffer;
    delete[] decBuffer;
    file.close();
}
void readMainFile(ifstream& filenames, char* buffer, int index, unsigned char key)
{
    char ch;
    if (!filenames.get(ch)) {
        if (index > 0) {
            *(buffer + index) = '\0';
            processFile(buffer, key);
        }
        return;
    }
    if (ch == '\n' || ch == '\r') {
        if (index > 0) {
            *(buffer + index) = '\0';
            processFile(buffer, key);
            readMainFile(filenames, buffer, 0, key);
        }
        else {
            readMainFile(filenames, buffer, 0, key);
        }
        return;
    }
    *(buffer + index) = ch;
    readMainFile(filenames, buffer, index + 1, key);
}
//----------------------main--------------------
/*#include "Header.h"
#include<iostream>
using namespace std;*/
/*int main()
{
    char password[8];
    cout << "Enter 8-character password:\n";
    getpassword(password);
    unsigned char key = findkey(password);

    ifstream filenames("filenames.txt");
    if (!filenames) {
        cout << "Cannot open filenames.txt\n";
        return 1;
    }

    char buffer[256];
    readMainFile(filenames, buffer, 0, key);
    filenames.close();
    return 0;
}*/


//-----------------------------------------------QUESTION 3---------------------------------
//----------MAIN----------
/*
int main()
{
    int size;
    char c1, c2;
    cout << "Enter the size: ";
    cin >> size;
    cout << "Enter first character: ";
    cin >> c1;
    cout << "Enter second character: ";
    cin >> c2;
    printDiamond(size, c1, c2);
    return 0;
}*/
//header------------------------------

void includespaces(int size, int i, int s = 1)
{
    if (s > (size - i)) return;
    cout << " ";
    includespaces(size, i, s + 1);
}

void includechars(int size, int i, char c1, char c2, int j = 1)
{
    if (j > (2 * i - 1)) return;
    if (j % 2 == 1)
        cout << c1;
    else
        cout << c2;
    includechars(size, i, c1, c2, j + 1);
}

void includetrailingspaces(int size, int i, int s = 1)
{
    if (s > (size - i)) return;
    cout << " ";
    includetrailingspaces(size, i, s + 1);
}

void includespaces2(int size, int i2, int j = 1)
{
    if (j > i2) return;
    cout << " ";
    includespaces2(size, i2, j + 1);
}

void includechars2(int size, int i2, char c1, char c2, int j2 = 1)
{
    if (j2 > (2 * (size - i2) - 1)) return;
    if (j2 % 2 == 1)
        cout << c1;
    else
        cout << c2;
    includechars2(size, i2, c1, c2, j2 + 1);
}

void includetrailingspaces2(int size, int i2, int s = 1)
{
    if (s > i2) return;
    cout << " ";
    includetrailingspaces2(size, i2, s + 1);
}

void lines2(int size, char c1, char c2, int i2)
{
    if (i2 > size - 1) return;

    // Determine starting character based on size parity
    char startChar, altChar;
    if (size % 2 == 1) {  // Odd size
        startChar = c1;
        altChar = c2;
    }
    else {  // Even size
        startChar = c2;
        altChar = c1;
    }

    includespaces2(size, i2);
    includechars2(size, i2, startChar, altChar);
    includetrailingspaces2(size, i2);
    cout << endl;
    lines2(size, c1, c2, i2 + 1);
}

void lines(int size, char c1, char c2, int i)
{
    if (i > size)
    {
        lines2(size, c1, c2, 1);
        return;
    }

    // Determine starting character based on size parity
    char startChar, altChar;
    if (size % 2 == 1) {  // Odd size
        startChar = c1;
        altChar = c2;
    }
    else {  // Even size
        startChar = c2;
        altChar = c1;
    }

    includespaces(size, i);
    includechars(size, i, startChar, altChar);
    includetrailingspaces(size, i);
    cout << endl;
    lines(size, c1, c2, i + 1);
}

void printDiamond(int size, char c1, char c2)
{
    lines(size, c1, c2, 1);
}
//----------------------------------------------------------------QUESTION 2------------------------------------


//-----------------------part 1--------------------------------------

int getLength(const char* str)
{
    int count = 0;
    if (str == nullptr)
    {
        count = 0;
    }
    else
    {
        while (*str != '\0')
        {
            ++count;
            str++;
        }
    }
    return count;
}

char* copyString(const char* str)
{
    if (str == nullptr)
    {
        return nullptr;
    }

    const char* temp = str;
    int count = 0;
    while (*temp != '\0')
    {
        ++count;
        temp++;
    }

    char* newmem = new char[count + 1];
    char* start = newmem;
    while (*str != '\0')
    {
        *newmem = *str;
        newmem++;
        str++;
    }
    *newmem = '\0';
    return start;
}

bool areEqual(const char* str1, const char* str2)
{
    if (str1 == nullptr || str2 == nullptr)
    {
        return false;
    }

    while (*str1 != '\0' && *str2 != '\0')
    {
        if (*str1 != *str2)
        {
            return false;
        }
        str1++;
        str2++;
    }

    return (*str1 == '\0' && *str2 == '\0');
}

char* concatenate(const char* str1, const char* str2)
{
    if (str1 == nullptr && str2 == nullptr)
    {
        return nullptr;
    }
    if (str1 == nullptr)
    {
        return copyString(str2);
    }
    if (str2 == nullptr)
    {
        return copyString(str1);
    }

    const char* temp1 = str1;
    int count1 = 0;
    while (*temp1 != '\0')
    {
        ++count1;
        temp1++;
    }

    const char* temp2 = str2;
    int count2 = 0;
    while (*temp2 != '\0')
    {
        ++count2;
        temp2++;
    }

    char* newmem = new char[count1 + count2 + 1];
    char* start = newmem;

    while (*str1 != '\0')
    {
        *newmem = *str1;
        newmem++;
        str1++;
    }
    while (*str2 != '\0')
    {
        *newmem = *str2;
        newmem++;
        str2++;
    }
    *newmem = '\0';

    return start;
}

void toUpperCase(char* str)
{
    if (str == nullptr) return;
    while (*str != '\0')
    {
        if (*str >= 'a' && *str <= 'z')
        {
            *str = *str - 32;
        }
        str++;
    }
}

void toLowerCase(char* str)
{
    if (str == nullptr) return;
    while (*str != '\0')
    {
        if (*str >= 'A' && *str <= 'Z')
        {
            *str = *str + 32;
        }
        str++;
    }
}

int findFirstOccurrence(const char* str, char target)
{
    if (str == nullptr) return -1;
    int count = 0;
    while (*str != '\0') {
        if (*str == target) {
            return count;
        }
        count++;
        str++;
    }
    return -1;
}

int findSubstring(const char* str, const char* substring)
{
    if (str == nullptr || substring == nullptr) return -1;
    if (*substring == '\0') return 0;

    int count = 0;
    while (*str != '\0') {
        const char* ptr1 = str;
        const char* ptr2 = substring;

        while (*ptr2 != '\0' && *ptr1 == *ptr2) {
            ptr1++;
            ptr2++;
        }

        if (*ptr2 == '\0') return count;

        str++;
        count++;
    }
    return -1;
}

char* reverseString(const char* str)
{
    if (str == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') {
        count++;
        temp++;
    }

    char* newstring = new char[count + 1];
    char* start = newstring;

    temp = str + count - 1;
    while (temp >= str) {
        *newstring = *temp;
        newstring++;
        temp--;
    }

    *newstring = '\0';

    return start;
}

char* removeCharacter(const char* str, char target) {
    if (str == nullptr) return nullptr;

    int count = 0;
    const char* ptr = str;
    while (*ptr != '\0') {
        count++;
        ptr++;
    }

    char* newstring = new char[count + 1];
    char* p = newstring;

    while (*str != '\0') {
        if (*str != target) {
            *p = *str;
            p++;
        }
        str++;
    }

    *p = '\0';
    return newstring;
}

char* substring(const char* str, int start, int length) {
    if (str == nullptr || start < 0 || length <= 0) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') {
        count++;
        temp++;
    }

    if (start >= count) return nullptr;

    char* newstring = new char[length + 1];
    char* p = newstring;
    int i = 0;
    int j = 0;
    temp = str;
    while (*temp != '\0') {
        if (i >= start && j < length) {
            *p = *temp;
            p++;
            j++;
        }
        i++;
        temp++;
    }
    *p = '\0';
    return newstring;
}

char* replaceChar(const char* str, char oldChar, char newChar) {
    if (str == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') {
        count++;
        temp++;
    }

    char* result = new char[count + 1];
    char* p = result;

    while (*str != '\0') {
        if (*str == oldChar) {
            *p = newChar;
        }
        else {
            *p = *str;
        }
        p++;
        str++;
    }
    *p = '\0';

    return result;
}

char* replaceSubstring(const char* str, const char* target, const char* replacement) {
    if (str == nullptr || target == nullptr || replacement == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') count++, temp++;

    int count1 = 0;
    temp = target;
    while (*temp != '\0') count1++, temp++;

    int count2 = 0;
    temp = replacement;
    while (*temp != '\0') count2++, temp++;

    int estSize = count * (count2 > count1 ? count2 : 1) + count + 1;
    char* result = new char[estSize];
    char* resultPtr = result;

    while (*str != '\0') {
        const char* tempS = str;
        const char* tempT = target;

        while (*tempS != '\0' && *tempT != '\0' && *tempS == *tempT) {
            tempS++;
            tempT++;
        }

        if (*tempT == '\0') {
            const char* rep = replacement;
            while (*rep != '\0') {
                *resultPtr = *rep;
                resultPtr++;
                rep++;
            }
            str = tempS;
        }
        else {
            *resultPtr = *str;
            resultPtr++;
            str++;
        }
    }

    *resultPtr = '\0';

    return result;
}

char* insertAt(const char* str, const char* toInsert, int position)
{
    if (str == nullptr || toInsert == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') {
        count++;
        temp++;
    }

    int count1 = 0;
    temp = toInsert;
    while (*temp != '\0') {
        count1++;
        temp++;
    }

    char* result = new char[count + count1 + 1];
    char* resPtr = result;

    int i = 0;
    temp = str;
    while (*temp != '\0') {
        if (i == position) {
            const char* ins = toInsert;
            while (*ins != '\0') {
                *resPtr = *ins;
                resPtr++;
                ins++;
            }
        }
        *resPtr = *temp;
        resPtr++;
        temp++;
        i++;
    }

    if (i == position) {
        const char* ins = toInsert;
        while (*ins != '\0') {
            *resPtr = *ins;
            resPtr++;
            ins++;
        }
    }

    *resPtr = '\0';

    return result;
}

char* deleteRange(const char* str, int start, int length) {
    if (str == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') {
        count++;
        temp++;
    }

    char* result = new char[count - length + 1];
    char* p = result;

    int i = 0;
    temp = str;
    while (*temp != '\0') {
        if (i < start || i >= start + length) {
            *p = *temp;
            p++;
        }
        i++;
        temp++;
    }
    *p = '\0';

    return result;
}

char* compress(const char* str)
{
    if (str == nullptr) return nullptr;

    int size = 0;
    const char* temp = str;
    while (*temp != '\0') size++, temp++;

    char* result = new char[size * 2 + 1];
    char* resPtr = result;

    int i = 0;
    while (i < size) {
        char current = *(str + i);
        int count = 1;

        while (i + 1 < size && *(str + i + 1) == current) {
            count++;
            i++;
        }

        *resPtr = current;
        resPtr++;

        *resPtr = count + '0';
        resPtr++;

        i++;
    }

    *resPtr = '\0';

    return result;
}

char* decompress(const char* str) {
    if (str == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') count++, temp++;

    char* result = new char[count * 10 + 1];
    char* resultPtr = result;

    int i = 0;
    while (*(str + i) != '\0') {
        int num = *(str + i + 1) - '0';

        for (int j = 0; j < num; j++) {
            *resultPtr = *(str + i);
            resultPtr++;
        }

        i += 2;
    }
    *resultPtr = '\0';

    return result;
}

char* rotateString(const char* str, int position) {
    if (str == nullptr) return nullptr;

    int count = 0;
    const char* temp = str;
    while (*temp != '\0') {
        count++;
        temp++;
    }

    char* result = new char[count + 1];

    if (count == 0) {
        *result = '\0';
        return result;
    }

    if (position < 0) position = count + (position % count);
    position = position % count;

    char* resPtr = result;
    int start = position;
    for (int k = 0; k < count; k++) {
        *resPtr = *(str + start);
        resPtr++;
        start = (start + 1) % count;
    }

    *resPtr = '\0';
    return result;
}

//----------------------------------------------PART 2------------------------------




char** mutationHistory;
int mutationCapacity;
int mutationCount;

void initializeVersionControl(int size)
{
    mutationHistory = new char* [size];
    mutationCapacity = size;
    mutationCount = 0;
}

void recordMutation(char* newTapeState, const char* operation, const char* inverseOp, const char* params)
{
    if (mutationCount >= mutationCapacity)
    {
        char** newmutation = new char* [mutationCapacity * 2];
        int i = 0;
        while (i < mutationCount)
        {
            *(newmutation + i) = *(mutationHistory + i);
            i++;
        }

        delete[] mutationHistory;

        mutationHistory = newmutation;
        mutationCapacity = mutationCapacity * 2;
    }

    char* copy = copyString(newTapeState);
    *(mutationHistory + mutationCount) = copy;

    ofstream file("mutation_log.txt", ios::app);
    file << mutationCount << "|" << operation << "|" << inverseOp << "|" << params << endl;
    file.close();

    mutationCount++;
}

void displayHistory()
{
    ifstream file("mutation_log.txt");
    string readfile;

    cout << "=== MUTATION HISTORY ===" << endl;

    while (getline(file, readfile))
    {
        string index = "";
        string operation = "";
        string inverse = "";
        string parameter = "";
        int partnumber = 1;

        int i = 0;
        while (i < readfile.length())
        {
            if (readfile[i] == '|')
            {
                partnumber++;
            }
            else
            {
                if (partnumber == 1)  index = index + readfile[i];
                if (partnumber == 2)  operation = operation + readfile[i];
                if (partnumber == 3)  inverse = inverse + readfile[i];
                if (partnumber == 4)  parameter = parameter + readfile[i];
            }
            i++;
        }
        cout << "[" << index << "] " << operation << " (inverse: " << inverse << ")" << endl;
    }
    file.close();
}

char* undoLastMutation()
{
    if (mutationCount <= 0) return nullptr;
    mutationCount--;
    char* result = copyString(*(mutationHistory + mutationCount));

    ifstream file("mutation_log.txt");
    char** lines = new char* [mutationCapacity];
    int lineCount = 0;
    string readfile;
    while (getline(file, readfile)) {
        *(lines + lineCount) = copyString(readfile.c_str());
        lineCount++;
    }
    file.close();

    char* oldLine = *(lines + (lineCount - 1));
    char* newLine = concatenate(oldLine, "|UNDONE");
    delete[] oldLine;
    *(lines + lineCount - 1) = newLine;

    ofstream outFile("mutation_log.txt");
    for (int i = 0; i < lineCount; i++) {
        outFile << *(lines + i) << endl;
    }
    outFile.close();

    for (int i = 0; i < lineCount; i++) {
        delete[] * (lines + i);
    }
    delete[] lines;
    return result;
}

char* revertToVersion(int index)
{
    if (index < 0 || index >= mutationCount) return nullptr;

    char* result = copyString(*(mutationHistory + index));
    ifstream file("mutation_log.txt");
    char** lines = new char* [mutationCapacity];
    int lineCount = 0;
    string readfile;

    while (getline(file, readfile)) {
        *(lines + lineCount) = copyString(readfile.c_str());
        lineCount++;
    }
    file.close();

    mutationCount = index + 1;

    for (int i = index + 1; i < lineCount; i++) {
        char* oldLine = *(lines + i);
        char* newLine = concatenate(oldLine, "|REVERTED");
        delete[] oldLine;
        *(lines + i) = newLine;
    }

    ofstream outFile("mutation_log.txt");
    for (int i = 0; i < lineCount; i++) {
        outFile << *(lines + i) << endl;
    }
    outFile.close();

    for (int i = 0; i < lineCount; i++) {
        delete[] * (lines + i);
    }
    delete[] lines;
    return result;
}

void permanentlyDeleteHistory()
{
    char* savedstate = copyString(*(mutationHistory + (mutationCount - 1)));

    for (int i = 0; i < mutationCount; i++) {
        delete[] * (mutationHistory + i);
    }

    *(mutationHistory + 0) = savedstate;
    mutationCount = 1;

    ofstream file("mutation_log.txt");
    file << "0|ORIGINAL_STATE||" << endl;
    file.close();
}

char* undoMultipleMutations(int* indicesToUndo, int count)
{
    for (int i = 0; i < count; i++)
    {
        if (*(indicesToUndo + i) < 0 || *(indicesToUndo + i) >= mutationCount)
            return nullptr;
    }

    int earliestindex = *(indicesToUndo + 0);

    for (int i = 1; i < count; i++)
    {
        if (*(indicesToUndo + i) < earliestindex)
        {
            earliestindex = *(indicesToUndo + i);
        }
    }

    char* result = nullptr;
    if (earliestindex > 0)
    {
        result = copyString(*(mutationHistory + earliestindex - 1));
        mutationCount = earliestindex;
    }
    else
    {
        result = copyString(*(mutationHistory + 0));
        mutationCount = 1;
    }

    ifstream file("mutation_log.txt");
    char** lines = new char* [mutationCapacity * 2];
    int lineCount = 0;
    string readfile;

    while (getline(file, readfile)) {
        *(lines + lineCount) = copyString(readfile.c_str());
        lineCount++;
    }
    file.close();

    for (int i = earliestindex; i < lineCount; i++) {
        char* oldLine = *(lines + i);
        char* newLine = concatenate(oldLine, "|UNDONE");
        delete[] oldLine;
        *(lines + i) = newLine;
    }

    ofstream outFile("mutation_log.txt");
    for (int i = 0; i < lineCount; i++) {
        outFile << *(lines + i) << endl;
    }
    outFile.close();

    for (int i = 0; i < lineCount; i++) {
        delete[] * (lines + i);
    }
    delete[] lines;

    return result;
}

void cleanupVersionControl()
{
    for (int i = 0; i < mutationCount; i++)
    {
        delete[] * (mutationHistory + i);
    }
    delete[] mutationHistory;
    mutationHistory = nullptr;
    mutationCapacity = 0;
    mutationCount = 0;
}
//-----------------------------------------------------------

